---
title: "Linkownia 08"
slug: "linkownia-08"
date: 2020-06-06T15:00:00+02:00
description: "Przegląd wartościowych linków od Nine Fives Labs [#8]"
tags: [devops, opensource, gitops, golang, build]
series: [linkownia]
authors: ["Wojtek Urbański"]
---

Cześć!

Czerwiec rozpoczynamy kolejną porcją ciekawostek na weekend.
GitOps nie znika z naszego radaru, bo uważamy, że z waszego też nie powinien jeśli chcecie oszczędzić sobie bólu przy pracy z Kubernetesem.
Na dokładkę dodamy też informacje o tym, jak zabrać się za powtarzalne budowanie aplikacji, nawet jeśli nie jesteśmy ich autorami. 
A na deser... Pokażemy, czy wytwarzanie oprogramowania dla rakiet to *rocket science*.
<!--more-->

### Paweł poleca: Prosty przepis babci Jadzi na GitOps

[GitOps Deployment and Kubernetes](https://medium.com/riskified-technology/gitops-deployment-and-kubernetes-f1ab289efa4b)

Niby wiemy, że jest coś takiego jak GitOps, ale czy wiemy od czego zacząć?
W linku znajdziecie prosty zestaw narzędzi, przydatnych do tego, by wprowadzić GitOps w swoje "życie".
Oprócz standardów jak GitHub Action K8s, Helm pojawia się ArgoCD, które jest sercem tłoczącym kod na zasoby w klastrze.
Czy da się prościej w GitOps?

### Wojtek poleca: powtarzalne buildy dla golanga z C

[Packaging LXD for Arch Linux](https://linderud.dev/blog/packaging-lxd-for-arch-linux/)

Czasami mam wrażenie, że umiejętność tworzenia powtarzalnych kompilacji zanika, albo przynajmniej jest coraz mniej poważana...
W świetle współczesnych języków programowania z systemami do zarządzania zależnościami, "przypinaniem" konkretnych wersji i całą
masą specjalnych systemów do kontroli wersji pakietów wydaje się, że już nie ma w tym temacie nic do roboty.
Otóż okazuje się, że także pracując nad budowaniem aplikacji w golangu można się nieźle napocić.
Autor powyższego posta pokazuje jak rozwiązał zagadkę połączenia golanga z C przy budowaniu LXD.

### Jarek poleca: Continuous deployment na orbicie

[Overview of how SpaceX approaches building their flight software](https://www.reddit.com/r/spacex/comments/7j69ue/overview_of_how_spacex_approaches_building_their/)

Oglądając start misji Dragon Crew Demo-2 zastanawiałem się jak wygląda proces CI/CD w tak krytycznych projektach.
Zainteresowanie wzrosło kilkukrotnie po tym, jak dowiedziałem się, że Boeing w trakcie testowej misji swojej kapsuły 
Starliner musiał [zainstalować łatkę oprogramowania w trakcie, gdy ta znajdowała się już na orbicie](https://spaceflightnow.com/2020/02/28/boeing-says-thorough-testing-would-have-caught-starliner-software-problems/).
Pod linkiem możecie znaleźć prezentację inżynierów SpaceX, którzy opowiadają jak wytwarza się u nich software do rakiet.
Prezentacja odbyła się dwa lata temu, ale każde uchylenie rąbka tajemnicy jest w tym wypadku cenne.

