---
title: "Sidecar czyli mutacje w Kubernetesie"
slug: "mutating_webhook"
date: 2020-05-27T10:00:00+02:00
description: "Pierwsze kroki w Dynamic Admission Control Kubernetesa"
tags: [kubernetes, python, dynamic admission control]
series: [breaking-kubernetes-network]
---

Gdy już wyjdziecie z bunkrów może was zaskoczyć, że mutacja nie jest jeszcze tak powszechna jak zapowiadały gry. 
Nic bardziej mylne, można mutować w Kubernetesie za pomoc Dynamic Admission Control. Przedstawię wam jak dodać sidecar z użyciem Flaska.

[Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) 
w skrócie są to webhooki, które można dodać w czasie runtime. Mamy dwa typy admission webhooks: walidacje (validating 
admission webhook) i mutacje (mutating admission webhook). Pierwszy weryfikuje nasz requesty, np czy są wszystkie wymagane `label`, albo 
czy ilość replik jest większa od minimalnej wartość. Drugi typ pozwala na zmiany requestu który potem trafia do etcd, np. może 
zmienić ilość replik jeśli jest za mała, albo dodać dodatkowy kontener do poda(`sidecar`). Przykładowo sidecarów możemy używać się
do monitoringu, zbierania logów czy też sevice meshu. 
Mamu wiele przykładowych projektów które używają sidecarów jak [Prometheus](https://prometheus.io/), [Fluentd](https://www.fluentd.org/) czy też  [Envoy](https://www.envoyproxy.io/)

<przerysować flow https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/>


Wymagania: 
 - [kind](https://github.com/kubernetes-sigs/kind).
 - Python 3.8
 - Flask


Przy generowaniu certifkatów pójdziemy na skróty i użyjemy skryptów z [tutoriala](https://github.com/morvencao/kube-mutating-webhook-tutorial/tree/master/deployment).
```bash
./webhook-create-signed-cert.sh --service mutate-webhook-svc --namespace default --secret mutate-webhook-secret
export CA_BUNDLE=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.ca\.crt}")
cat ./mutate_admission.yaml | ./kube-mutating-webhook-tutorial/deployment/webhook-patch-ca-bundle.sh > ./mutate_admission_ca.yaml 
```

Pierwsze zasoby to dość standardowy deployment z serwisem, który przyłącza secret z certyfkatami które były stworzone w 
startowej komendzie poprzedniego kroku. 
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mutate-webhook
spec:
  selector:
    matchLabels:
      run: mutate-webhook
  replicas: 1
  template:
    metadata:
      labels:
        run: mutate-webhook
    spec:
      containers:
      - name: mutate-webhook
        image: mutate
        imagePullPolicy: Never
        ports:
        - containerPort: 443
        volumeMounts:
        - name: webhook-certs
          mountPath: /certs
          readOnly: true
      volumes:
      - name: webhook-certs
        secret:
          secretName: mutate-webhook-secret
---
apiVersion: v1
kind: Service
metadata:
  name: mutate-webhook-svc
  labels:
    run: mutate-webhook
spec:
  ports:
  - port: 443
    protocol: TCP
  selector:
    run: mutate-webhook
```
Oczywiście Dockerfile...
```bash
FROM python:alpine3.11
RUN pip install flask jsonpatch
COPY mutate.py mutate.py
CMD python mutate.py
```

Tworzę prosty webook w Pythonie z użyciem Flaska. Funcja `add_side_car_webhook` z dekoratorem `route` z wyznaczoną ścieżką, 
która potem będzie użyta w definicji zasobu który opis ten webhook w Kuberensie. 
Na jego podstawie k8s puka do naszego webhook z requestem tworzącym 
pod, my go odbieramy i odsyłamy zmiany. W odpowiedzi musimy podać wersje api(`apiVersion`), typ(`kind`) oraz request. 
Wewnątrz requestu dajemy pozwolenie (`allow`) na dalsze jego przetwarzanie przez k8s wraz z uid(`uid`) odebranego request. 
A przede wszystkim podajemy typ (`patchType`) oraz naszą zmianę (`patch`). 
Nasza zmiana musi być odpowiednio kodowana, ale aż tak czarną magią nie będziemy się dziś zajmować. 

Jak zmutować naszego poda? Jest to dość proste w `patch` musimy podać: jaką operację(`op`) chcemy wykonać, 
ścieżkę(`path`) oraz wartość(`value`). Chcemy dodać sidecar do konteneru z bazowego requestu, dlatego odczytujemy istniejącą
zawartość `request_info['request']['object']['spec']['containers']` i dodajemy naszą konfigrurację kontenera.  
```python
import base64
import jsonpatch

from flask import Flask, request, jsonify


admission_controller = Flask(__name__)
container_path = "/spec/containers"
side_car_config = {"name": "busybox2",
                   "image": "busybox",
                   "command": ["sleep", "3600"],
                   "imagePullPolicy": "IfNotPresent"}


@admission_controller.route('/mutate/pods', methods=['POST'])
def add_side_car_webhook():
    request_info = request.get_json()
    containers = request_info['request']['object']['spec']['containers']
    containers.append(side_car_config)
    return admission_response_patch(True, 
                                    request_info['request']['uid'], 
                                    "Adding side car", 
                                    json_patch=jsonpatch.JsonPatch([{"op": "add",
                                                                     "path": container_path,
                                                                     "value": containers}]))


def admission_response_patch(allowed, uid, message, json_patch):
    base64_patch = base64.b64encode(json_patch.to_string().encode("utf-8")).decode("utf-8")
    return jsonify({"apiVersion": "admission.k8s.io/v1",
                    "kind": "AdmissionReview",
                    "response": {"allowed": allowed,
                                 "status": {"message": message},
                                 "uid": uid,
                                 "patchType": "JSONPatch",
                                 "patch": base64_patch}})

if __name__ == '__main__':
    admission_controller.run(host='0.0.0.0', port=443, ssl_context=("/certs/cert.pem", "/certs/key.pem"))

```

Teraz musimy wyznaczyć, do którego serwisu i jakiej ścieżki użyć, tworzymy zasób MutatingWebhookConfiguration z takim opisem.
ClientConfig określa do jakiego serwisu, pod jaką ścieżkę oraz z jakim certyfikatem k8s wyśle request do przetworzenia. 
`Rules` decydują, które requesty tam trafią, możemy wybierać requesty na podstawie grupy albo wersji api, zasobu, czy też operacji.  

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: mutating-webhook
  namespace: test
webhooks:
  - name: test.mutate.com
    failurePolicy: Fail
    admissionReviewVersions: ["v1"]
    sideEffects: None
    clientConfig:
      service:
        name: mutate-webhook-svc
        namespace: default
        path: /mutate/pods
      caBundle: ${CA_BUNDLE}
    rules:
      - apiGroups:
          - "*"
        resources:
          - "pods"
        apiVersions: ["v1"]
        operations:
          - CREATE
```

Tworzymy zasoby:
0. Tworzymy certyfikaty
```bash
./webhook-create-signed-cert.sh --service mutate-webhook-svc --namespace default --secret mutate-webhook-secret
export CA_BUNDLE=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.ca\.crt}")
cat ./mutate_admission.yaml | ./kube-mutating-webhook-tutorial/deployment/webhook-patch-ca-bundle.sh > ./mutate_admission_ca.yaml 
```
1. Tworzymy obraz
```bash
docker build . -t mutate
```
2. Wypychamy do kinda
```bash
kind load docker-image mutate
```
3. Tworzymy webook z serwisem
```bash
kubectl apply -f webhook.yaml
```
4. Tworzymy poteżną konfigurację mutującego webhooka
```bash
kubectl apply -f mutate_admission_ca.yaml
```
5. Mutujemy busyboxa
```bash
kubectl apply -f box.yaml
```

**Dostajemy pod po mutacji, który posiada sidecar**
```bash
# kubectl get pods
NAME                             READY   STATUS        RESTARTS   AGE
busybox1                         2/2     Running       0          46s
mutate-webhook-bc869859d-888lc   1/1     Running       0          47s
# kubectl describe pod busybox1 
Name:         busybox1
Namespace:    default
Priority:     0
Node:         kind-control-plane/172.18.0.2
Start Time:   Sat, 09 May 2020 08:20:04 +0200
Labels:       app=busybox1
Annotations:  Status:  Running
...
Containers:
  busybox:
    Container ID:  containerd://e65663fceb302609ea247b7679bdce99d289f533004f5698607dc0dcfac53a7d
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:a8cf7ff6367c2afa2a90acd081b484cbded349a7076e7bdf37a05279f276bc12
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Sat, 09 May 2020 08:20:05 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-h2v64 (ro)
  busybox2:
    Container ID:  containerd://79be7e229ebac9ce5cd22372f9cb34f3a82a4fc5da4e331ee94b7d9379b46369
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:a8cf7ff6367c2afa2a90acd081b484cbded349a7076e7bdf37a05279f276bc12
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      3600
    State:          Running
      Started:      Sat, 09 May 2020 08:20:05 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-h2v64 (ro)
...
Events:
  Type    Reason     Age        From                         Message
  ----    ------     ----       ----                         -------
  Normal  Scheduled  <unknown>  default-scheduler            Successfully assigned default/busybox1 to kind-control-plane
  Normal  Pulled     112s       kubelet, kind-control-plane  Container image "busybox" already present on machine
  Normal  Created    112s       kubelet, kind-control-plane  Created container busybox
  Normal  Started    112s       kubelet, kind-control-plane  Started container busybox
  Normal  Pulled     112s       kubelet, kind-control-plane  Container image "busybox" already present on machine
  Normal  Created    112s       kubelet, kind-control-plane  Created container busybox2
  Normal  Started    112s       kubelet, kind-control-plane  Started container busybox2

```

Sukces, mamy nasz side car. Dynamic Admission Control w łatwy sposób daje wiele możliwość do dynamicznych zmian. 

W czym może być pomocny taki sidecar? Postaram się przedstawić w najbliższym czasie. 

Cały kod znajdziecie pod tym linkiem https://github.com/ninefiveslabs/side_car_mutate

Linki:
 - https://kind.sigs.k8s.io/
 - https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/
 - https://medium.com/analytics-vidhya/how-to-write-validating-and-mutating-admission-controller-webhooks-in-python-for-kubernetes-1e27862cb798
 - https://github.com/morvencao/kube-mutating-webhook-tutorial/tree/master/deployment