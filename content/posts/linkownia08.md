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

### Misiek poleca: Continuous Delivery of Ideas

[What Will The Next 10 Years Of Continuous Delivery Look Like?](https://www.youtube.com/watch?v=FVEWdatM8Uk)

Prezentacja prowadzona przez dwóch autorów książki `Continuous Delivery`, w
której przedstawiają jak technicznie tworzyć proces CI/CD.

W prezentacji Dave Farley i Jez Humble skupiają się na szerszym pojęciu CD -
coś, czego ich zdaniem zabrakło w książce, którą napisali 10 lat temu.

Roztaczają wizję CD jako ogólnego systemu 'dowożenia' - od idei do gotowego
produktu, której właściwie nie da się osiągnąć bez kulturalnej zmiany
organizacji, a w szczególności nastawienia warstwy zarządzającej.

Przedstawiają że trzeba rozróżnić 'Product design' i 'Product delivery' -
pierwszy to worek pomysłów weryfikowany poprzez zastosowanie metody naukowej,
a drugi to narzędzia wspomagające ten pierwszy (i o tym jest książka).

Nagranie z konferencji DeliveryConf 2020 - to było w styczniu w Seattle, czyli
prawie 6 miesięcy temu....
Zachęcam do zajrzenia do reszty nagrań, bo jest bez typowego wciskania
marketingowego kitu - [DeliveryConf 2020 playlist](https://www.youtube.com/playlist?list=PLir_MEJB8aJiRUrlU6MFOvUsF2X0lAaQP).

