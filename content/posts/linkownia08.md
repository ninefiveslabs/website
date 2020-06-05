---
title: "Linkownia 08"
slug: "linkownia-08"
date: 2020-06-05T18:00:00+02:00
description: "Przegląd wartościowych linków od Nine Fives Labs [#8]"
tags: [devops, opensource, gitops, golang, build]
series: [linkownia]
authors: ["Wojtek Urbański"]
---

Cześć!
Czerwiec rozpoczynamy kolejną dawką ciekawostek na weekend.
GitOps nie znika z naszego radaru, bo uważamy, że z waszego też nie powinien.
Na dokładkę dodamy też informacje o tym, jak zadbać o powtarzalne budowanie aplikacji, nawet jeśli nie jesteśmy ich autorami. 
A na deser...
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

