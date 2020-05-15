---
title: "Linkownia 05"
slug: "linkownia05"
date: 2020-05-15T16:00:00+02:00
description: "Przegląd wartościowych linków od Nine Fives Labs [#5]"
tags: [devops, opensource, gitops, mikroserwisy]
series: [linkownia]
authors: ["Wojtek Urbański"]
---

W dzisiejszym odcinku "linkowni" znajdziecie zbiór informacji o architekturze aplikacji jednego z królów mikroserwisów - Netfliksa, ale także dowiecie się jak zarządzać przechowywanymi w repozytoriach poświadczeniami w prosty, ale bezpieczny sposób przy użyciu narzędzia Mozilla SOPS. O automatyzacji napisano już wiele, ale system GitHub Actions, o którym mówi ostatni artykuł, zdecydowanie jest nowością, którą można wykorzystywać na wiele sposobów. Jak wiele? przeczytajcie sami!
<!--more-->

### Paweł poleca: Poznaj architekturę serwisu, który ratuje świat

[A Design Analysis of Cloud-based Microservices Architecture at Netflix](https://medium.com/swlh/a-design-analysis-of-cloud-based-microservices-architecture-at-netflix-98836b2da45f)

W dobie wszechobecnego lockdownu istnieje organizacja, która ratuje nas przed szaleństwem. Jest to Netflix, czyli największy serwis streamingowy. W powyższym linku możecie przeczytać jak aktualnie wygląda architektura Netfliksa. Dodatkowo znajdziecie trochę informacji na temat ewolucji  przejścia na mikroserwisy, które trwało prawie 10 lat. 

### Wojtek poleca: Zarządzanie sekretami przy użyciu Mozilla SOPS

[HashiCorp Vault is Overhyped, and Mozilla SOPS with KMS and Git is Massively Underrated](https://oteemo.com/2019/06/20/hashicorp-vault-is-overhyped-and-mozilla-sops-with-kms-and-git-is-massively-underrated/)

Wiele osób na dźwięk słów "przechowywanie haseł" myśli od razu o Hashicorp Vault - ale czy to rzeczywiście jedyny słuszny sposób na przechowywanie tajnych danych w środowisku programistycznym? Mozilla SOPS jest świetnym, małym narzędziem, pozwalającym na zarządzanie tajnymi informacjami i przechowywanie ich np. w repozytorium gitowym (choć nie tylko!) i korzysta z kilku technologii zapewniających bezpieczeństwo - nie tylko z popularnego GPG, ale także usług typu Key Management Service (KMS) od głównych dostawców chmurowych. 

### Jarek poleca: GitHub Actions dla średniozaawansowanych

[Deploy your pull requests with GitHub Actions and GitHub Deployments](https://sanderknape.com/2020/05/deploy-pull-requests-github-actions-deployments/)

GitHub Actions są znane jako platforma CI, dzięki której w prosty sposób można skorzystać z gotowych, stworzonych przez społeczność skryptów do testowania kodu. Autor artykułu opisuje swój pomysł na trochę bardziej zaawansowane użycie Actions - wywoływane komentarzami pod pull requestem wdrożenia aplikacji na różne środowiska. Sprytnie łącząc ze sobą wiele operacji wywołuje API i uruchamia skrypty. Gdy przejrzymy końcowy plik konfiguracyjny z repozytorium, zaczyna to niebezpiecznie przypominać ansiblowego potworka i programowanie w YAMLu. Ciekawe, ale polecam ostrożność przy naśladowaniu.
