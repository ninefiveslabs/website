---
title: "Linkownia 04"
slug: "linkownia04"
date: 2020-05-08T15:30:00+02:00
description: "Przegląd wartościowych linków od Nine Fives Labs [#4]"
tags: [devops, kubernetes, opensource, gitops]
series: [linkownia]
---

Cześć, w dzisiejszym zbiorze linków trochę o związkach kryzysu z otwartym oprogramowaniem, continuous deployment w stylu GitOps i wydajności aplikacji na Kubernetesie. Jeśli przejrzycie wszystko i nadal będzie Wam mało, możecie też sprawdzić inne wpisy z [tej serii]({{< ref "/series/linkownia" >}}).

### Paweł poleca: Trzecia fala, na szczęście opensource'u.

[The third wave of open source migration](https://blog.tidelift.com/the-third-wave-of-open-source-migration)

W dobie nadchodzącego kryzysu firmy zaczynają oszczędzać, a najłatwiej zrezygnować z usług, które można zastąpić darmowym, otwartym kodem. Krótki tekst, opowiadający o tym, jak poprzednie kryzysy gospodarcze znacznie przyspieszyły pracę w projektach opensource. Wpis jest interesujący mimo niewielkiej ilości technikaliów. Może faktycznie rozwój otwartych rozwiązań będzie pozytywnym aspektem tych czasów.

### Wojtek poleca: Od zera do modelu GitOps przy użyciu dużej liczby modnych narzędzi

[A Complete Step by Step Guide to Implementing a GitOps Workflow with Flux](https://managedkube.com/gitops/flux/weaveworks/guide/tutorial/2020/05/01/a-complete-step-by-step-guide-to-implementing-a-gitops-workflow-with-flux.html)

Zestaw narzędzi wykorzystany w powyższym poradniku może wydawać się nieco nadmiarowy (Kustomize, Helm 3, Flux), ale twórcy przekonują, że wszystko jest OK i to tylko ułatwia życie. Abstrahując od tego, GitOps wydaje się aktualnie być Świętym Graalem zarządzania Kubernetesem i ja do tej pory nie znalazłem żadnego tak dokładnego poradnika, który pokazałby drogę do GitOps w projekcie typu greenfield. Polecam!

### Jarek poleca: Uruchamianie aplikacji Javowych na Kubernetesie: jak przygotować im dobre środowisko

[Java Application Optimization on Kubernetes](https://medium.com/faun/java-application-optimization-on-kubernetes-on-the-example-of-a-spring-boot-microservice-cf3737a2219c)

Artykuł opisuje jak dobrze skonfigurować deploymenty Javowe na Kubernetesie pod kątem przydziału zasobów, liveness probes itp. Nie ma tam za dużo odkryć, ale jest pokazane, że k8s to szczególne środowisko wdrożeniowe i trzeba przemyśleć kilka szczegółów zanim zacznie się tam uruchamiać aplikacje - nie jest to proste uruchomienie procesu, a nawet nie jest bezpośrednim przeniesieniem z kontekstu lokalnych kontenerów (np. na Dockerze w środowisku deweloperskim).
