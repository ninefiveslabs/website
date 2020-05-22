---
title: "Linkownia 06"
slug: "linkownia06"
date: 2020-05-22T18:00:00+02:00
description: "Przegląd wartościowych linków od Nine Fives Labs [#6]"
tags: [devops, opensource, kubernetes, kontenery]
series: [linkownia]
authors: ["Jarek Łukow"]
---

Linkownia w 21. tygodniu roku - intensywne słońce pada na monitor po przebiciu się przez gęsty labirynt nieporuszonych nawet najlżejszym powiewem wiatru liści na drzewach. A na monitorze: kontenery, dynamiczne modyfikacje zasobów w locie i imperatywne generowanie deklaratywnych konfiguracji. Zapraszamy do lektury!
<!--more-->

### Paweł poleca: Mutacje i walidacje w Kubernetesie

[How To Write Validating and Mutating Admission Controller Webhooks in Python for Kubernetes](https://medium.com/analytics-vidhya/how-to-write-validating-and-mutating-admission-controller-webhooks-in-python-for-kubernetes-1e27862cb798)

Czy Twoim zasobom czegoś brakuje? A może potrzebujesz je rozszerzyć na etapie tworzenia? Admission controllery pozwolą na to, a pod linkiem znajdziecie łatwy przepis jak zacząć z użyciem Flaska.

### Jarek poleca: Imperatywnie deklaratywnie

[AWS’ cdk8s, a Dev-Friendly Alternative to YAML for Managing Kubernetes Clusters]([https://thenewstack.io/aws-cdk8s-a-dev-friendly-alternative-to-yaml-for-managing-kubernetes-clusters/](https://thenewstack.io/aws-cdk8s-a-dev-friendly-alternative-to-yaml-for-managing-kubernetes-clusters/))

Wraz z rozwojem automatyzacji w repozytoriach kodu pojawia się coraz więcej linii w plikach YAML i JSON.
Osoby utrzymujące te zbiory konfiguracji wiedzą, że dość szybko osiąga się poziom, w którym tworzenie plików w formacie czysto tekstowym przestaje być możliwe - mam na myśli podstawianie wielokrotnie tych samej wartości, powtarzalne struktury plików itp.
Dlatego popularność zdobyły różne sposoby na generowanie tekstu z szablonów, jak na przykład Jinja2 w Ansible czy Go templates w Helmie.
Ale skoro YAML to parsowalna struktura - mapy map napisów i liczb - podejście tekstowe wydaje się być technologicznym zacofaniem.
Sam wiele razy musiałem diagnozować błędy spowodowane złymi wcięciami w plikach YAML generowanych z szablonu.
Swoje rozwiązanie tego problemu proponuje AWS, publikując narzędzie cdk8s (przeznaczone specjalnie dla Kubernetesa, jego starszym bratem jest projekt CDK, który konwertuje hierarchię obiektów języka programowania na manifesty CloudFormation do wdrożeń na chmurę AWS).
Jest to bibliotekoframework do pisania skryptów, których wynikiem są struktury danych konwertowane potem na mapy w YAMLu.
Można więc tworzyć listy w pętli, definiować zmienne i używać schematów obiektowych do powtarzalnych elementów.
Wydaje się, że to świetny pomysł, pod warunkiem, że użytkownicy nie będę nadużywać tej ogromnej siły wyrazu do tworzenia nadmiernie skomplikowanych generatorów.

PS Dla porównania możesz rzucić okiem na wcześniejsze projekty realizujące podobną koncepcję: [Ballerina]([https://ballerina.io/](https://ballerina.io/)) i [ksonnet]([https://github.com/ksonnet/ksonnet/blob/master/docs/concepts.md](https://github.com/ksonnet/ksonnet/blob/master/docs/concepts.md)) (martwy).

### Wojtek poleca: Od zera do kontenera (według linuksowego developera)

[Linux containers in a few lines of code](https://zserge.com/posts/containers/)

Docker wykorzystuje izolację na wielu płaszczyznach, ale czasami ciężko sobie wyobrazić na czym dokładnie ona polega. Jeszcze trudniej zauważyć, że to wszystko... jest już dostępne w naszym systemie!
Ten post przedstawia jak w kilkudziesięciu liniach kodu uruchomić kontener dockerowy `busybox` bez Dockera.
