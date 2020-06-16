---
title: "Jsonnet: konfiguracja jako kod, a nie szablony"
slug: "jsonnet-konfiguracja-jako-kod"
date: 2020-06-20T11:00:00+02:00
tags: [kubernetes, jsonnet, devops]
series: [jsonnet]
authors: ["Wojtek Urbański"]
---

W epoce chmurowej, jak lubię nazywać dzisiejsze czasy w świecie IT, króluje podejście zarządzania infrastrukturą,
aplikacjami i dowolnymi procesami w postaci kodu. Niestety wiąże się to z mniej przyjemną stroną medalu, czyli
"programowaniem w YAML-ach". Opowiem o tym, jak uniknąć tego problemu.
<!--more-->
Jeśli szukasz bezpośredniego wprowadzenia do świata języka Jsonnet, to przeskocz [dwa akapity](#jsonnet), bo najpierw będę
opowiadał trochę historyjek.

### Kod jako nie-kod

Warto mieć na uwadze, że administratorzy, czy też tak zwani DevOpsi, często wcale nie przepadają za programowaniem, a co
za tym idzie - bywa, że rozwiązania infrastruktury-jako-kodu niewiele mają wspólnego z prawdziwym kodem. Zwykle są to
raczej ustrukturyzowane opisy modelu danych, reprezentującego stan, który chcemy osiągnąć w naszej infrastrukturze. Taki
opis może zostać przedstawiony na przykład w formacie [JSON](https://www.json.org/), czy [YAML](https://yaml.org/)
rzadziej [TOML](https://github.com/toml-lang/toml) czy, znany
szczególnie z Windowsa, [INI](https://en.wikipedia.org/wiki/INI_file), a następnie zamieniony na odpowiednie operacje
przez aplikację taką jak Ansible czy api-server Kubernetesa. Ale niezależnie od tego, jaki format wybierzemy,
najczęściej pliki konfiguracyjne da się przedstawić w sposób równoważny, jako dowolny inny rodzaj reprezentacji. 

Głównym problemem zarządzania infrastrukturą przy użyciu zestawu plików tekstowych (to smutna prawda, w YAML-u
się nie programuje) jest to, jak podmienić w locie pewne dane, powtórzyć struktury, czy ustalić określone wartości na
podstawie innych. Oczywistym podejściem jest korzystanie z szablonów, bo przecież operujemy na plikach tekstowych. Zatem
tworzymy szablony (np. korzystając z języka szablonów w Helmie, czy Jinja2 w Ansible, a bardziej hardkorowi zawodnicy
używają `sed` i `envsubst`), które na podstawie dostarczonych do nich danych, tworzą pliki tekstowe, reprezentujące
zaplanowane przez nas dane... 

Przepraszam, co?

### Dane z plików tekstowych stworzonych z danych zapisanych w pliku tekstowym

Mogłem coś pokręcić, ale chyba to, co tu robimy nie ma specjalnego sensu? Skupmy się na przykładzie Helma, w którym
tworzymy pliki YAML, bazując na odpowiednich szablonach wypełnionych danymi. Zapisując dane w pliku tekstowym musimy
wziąć pod uwagę odpowiednie poziomy zagnieżdżenia, usuwanie białych znaków, typy danych, które są akceptowane przez
docelowe API Kubernetesa i całą masę innych problemów. Tylko dlatego, że chcemy w prosty sposób wstrzyknąć przygotowane
przez nas dane do pliku tekstowego, które je reprezentuje... Ach, no i nie zapominajmy, że ten YAML ostatecznie staje się
JSON-em podczas przesyłania do API. To się nie skaluje zbyt dobrze. Ostatecznie chyba każdy po krótkim czasie wyrabia
sobie wstręt do JSON-ów.

![Json? Nyet!](json-nyet.png)

Według mojej (subiektywnej) opinii, szablony do generowania YAML-a są złe, a tworzenie w ten sposób JSON-ów byłoby jeszcze gorsze (przecinki na końcu obiektów!).
Gdyby tylko była inna droga... Na szczęście jest! I to zapewne niejedna, ale ja dzisiaj skupię się na wybranej przeze
mnie, mianowicie na Jsonnecie.

### Jsonnet

[Jsonnet](https://jsonnet.org/) jest językiem tworzenia ustrukturyzowanych danych, który doskonale sprawdza się w
zarządzaniu rozległymi, ale powtarzalnymi konfiguracjami czy definicjami. Stworzony został jako tzw. "20% project" w
2014 r. w Google, ale został ciepło przyjęty w wielu innych organizacjach i jest wykorzystywany na przykład w projekcie
[prometheus-operator](https://github.com/coreos/prometheus-operator) do generowania wszystkich statycznych plików YAML,
których używa pewnie całkiem spora grupa administratorów. Jsonnet wykorzystuje i rozszerza dobrze znaną składnię
wspominanego już JSON-a, dodając na przykład komentarze i pozwalając na **stawianie przecinka po ostatnim polu w
mapie/obiekcie** (w JSON-ie nie można tego robić, serio).

Funkcjonalności obsługiwanych przez Jsonnet jest całkiem sporo i możesz je poznać, korzystając z [interaktywnych
samouczków](https://jsonnet.org/learning/tutorial.html), skupię się zatem na wymienieniu kilku moich ulubionych:

- Przecinki po ostatnim elemencie w tablicy/mapie i komentarze - naprawdę, to są genialne funkcjonalności.
- Podstawowe konstrukcje programistyczne - pętle, instrukcje warunkowe, rozszerzanie obiektów, standardowa biblioteka
  funkcji.
- Definiowanie patchy, które są nakładane na inne obiekty, pozwalając na dokładną
  kontrolę łączenia obiektów (deep merge). Jako przykład weźmy obiekt,
  który zmienia liczbę replik w deploymencie na Kubernetesie bez naruszania
  innych pól mapy:
  ```
  local replica_patch = {
    spec+: {
      replicas: 3
    }
  };
  
  deployment + replica_patch
  ```
- Definiowanie funkcji, które mogą pełnić rolę "fabryk obiektów" - na podstawie otrzymanych argumentów zwracać
  odpowiednie struktury, co pozwala na zmniejszenie powtórzeń w kodzie:
  ```
  local new_app(app_name, version='1.0') {
    name: app_name,
    version: version,
    schema: "v1",
  }
  ```
- Możliwość tworzenia i importowania bibliotek:
  ```
  local my_lib = import "library.libsonnet";
  
  my_lib.my_function(with, arguments)
  ```

Przykładem połączenia niektórych z tych rozwiązań niech będzie zamiana poniższego kodu, definiującego kilka deploymentów w Kubernetesie...

```jsonnet
local new_deployment(new_name) = {
  apiVersion: "apps/v1",
  kind: "Deployment",
  metadata: {
    name: new_name,
  },
  spec: {
    replicas: 1,
    template: {
      metadata: {
        labels: {
          app: new_name,
        },
      },
      spec: {
        containers: [], // tu coś trzeba dodać... :)
      },
    },
  },
};

local namespace_patch(ns) = {
  metadata+: {
    namespace: ns
  }
};

local counter = std.range(1,3);

[
  new_deployment("deployment-%d" % i) +
  namespace_patch("namespace-%03d" % i) + { spec+: {replicas: i }}
  for i in counter
]
```

...na następującą tablicę, przy użyciu polecenia `jsonnet example.jsonnet`:

```json
[
   {
      "apiVersion": "apps/v1",
      "kind": "Deployment",
      "metadata": {
         "name": "deployment-1",
         "namespace": "namespace-001"
      },
      "spec": {
         "replicas": 1,
         "template": {
            "metadata": {
               "labels": {
                  "app": "deployment-1"
               }
            },
            "spec": {
               "containers": [ ]
            }
         }
      }
   },
   {
      "apiVersion": "apps/v1",
      "kind": "Deployment",
      "metadata": {
         "name": "deployment-2",
         "namespace": "namespace-002"
      },
      "spec": {
         "replicas": 2,
         "template": {
            "metadata": {
               "labels": {
                  "app": "deployment-2"
               }
            },
            "spec": {
               "containers": [ ]
            }
         }
      }
   },
   {
      "apiVersion": "apps/v1",
      "kind": "Deployment",
      "metadata": {
         "name": "deployment-3",
         "namespace": "namespace-003"
      },
      "spec": {
         "replicas": 3,
         "template": {
            "metadata": {
               "labels": {
                  "app": "deployment-3"
               }
            },
            "spec": {
               "containers": [ ]
            }
         }
      }
   }
]
```

Szybciej? No szybciej. I bez tekstowych szablonów.

Powyższe funkcje, w połączeniu z absolutną powtarzalnością danych generowanych z kodu Jsonneta powodują, że doskonale
nadaje się on do tworzenia złożonych struktur, takich jak manifesty kubernetesowe. 

W tym właśnie celu powstał kiedyś projekt [ksonnet](https://github.com/ksonnet/ksonnet), który jednak został 
[wstrzymany po przejęciu Heptio przez VMWare](https://tanzu.vmware.com/content/blog/welcoming-heptio-open-source-projects-to-vmware). 
Dostarczał on bibliotek generowanych na podstawie definicji OpenAPI (Swaggera), które pozwalały na składanie z kodu plików
zrozumiałych dla Kubernetesa. Mimo braku dalszego rozwoju, działa na tyle dobrze, że nadal jest on wykorzystywany
przez zespół wspominanego wcześniej prometheus-operatora.

Na szczęście ksonnet nie jest jedynym produktem wykorzystującym Jsonnet do wspomagania pracy z Kubernetesem. Ze ściętej
głowy hydry wyrosło wiele kolejnych, a i nie zapominajmy, że świat nie kończy się na Kubernetesie i pełen jest
ustrukturyzowanych reprezentacji danych.

### Kto z tego korzysta?

Trzymając się jeszcze nieco tematu Prometheusa i monitoringu, warto wspomnieć o projekcie
[grafonnet](https://github.com/grafana/grafonnet-lib). Służy on do
deklaratywnego opisu widoków (dashboardów) dla Grafany i jest wykorzystywany przez prometheus-operatora do tworzenia
podstawowych dashboardów.

Oczywiście zarządzanie Kubernetesem przy pomocy Jsonneta dalej jest tematem aktualnym i istnieją narzędzia takie jak
[splunk/qbec](https://github.com/splunk/qbec) czy [grafana/tanka](https://github.com/grafana/tanka), które pozwalają na
stosowanie plików w tym formacie do definiowania zasobów w Kubernetesie. Nie ma chyba sensu ukrywać, że kolejny artykuł w
serii, którą właśnie czytasz będzie mówił o tych narzędziach.

Ostatnim z moich przykładów wykorzystania Jsonneta w "prawdziwym świecie" niech będzie biblioteka do zarządzania Spinnakerem -
[Sponnet](https://github.com/spinnaker/sponnet) (chyba zaczynam widzieć jakiś wzorzec nazewnictwa...), która znacząco ułatwia tworzenie struktur wykorzystywanych w tym kombajnie do orkiestracji. Bezwstydnie wcisnę
też tutaj link do współtworzonego przeze mnie projektu, który opiera się o wykorzystanie między innymi właśnie Jsonneta do
zarządzania konfiguracją Spinnakera - [codilime/floodgate](https://github.com/codilime/floodgate). W nagraniu poniżej
możesz zobaczyć, jak tworzę definicję konfiguracji w Jsonnecie i jak znacząco ułatwia on pracę z plikami
konfiguracyjnymi (start ok. 38 minuty).

{{< youtube id="66S5LFM12JQ" start="2330" >}}

### Podsumowanie

Jestem fanem Jsonneta. Zacząłem go wykorzystywać przy okazji tworzenia projektu floodgate, ale teraz korzystam z niego
do zarządzania zasobami na Kubernetesie i przyznaję, że jest to jedno z narzędzi, które odmieniło mój sposób
"programowania konfiguracji".

W kolejnych postach tej serii pokażę bardziej szczegółowo, jak wykorzystać narzędzie qbec do zarządzania Kubernetesem, a
także jak podejść do utrzymywania zależności, bo jak wiadomo za kodem krok w krok podąża piekiełko zarządzania
bibliotekami...
