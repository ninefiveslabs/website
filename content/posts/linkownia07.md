---
title: "Linkownia 07"
slug: "linkownia07"
date: 2020-05-29T18:00:00+02:00
description: "Przegląd wartościowych linków od Nine Fives Labs [#6]"
tags: [devops, opensource, kubernetes, kontenery]
series: [linkownia]
authors: ["Paweł Kopka"]
---

Czy w pod koniec drugie dekadzie XXI wieku nadal potrzebujemy make? Jak często duże firmy korzystają z motorów? Czy jesteś bardziej lewicowy, czy prawicowy, jeśli chodzi o testy? Gdzie jest Wally? Linki z tym tygodni pomogą wam odpowiedzieć na te pytania, no może nie na wszystkie.
<!--more-->

### Wojtek poleca: make na miarę XXI wieku

[Using make and git diff for a simple and powerful test harness](https://chrismorgan.info/blog/make-and-git-diff-test-harness/)

Co tu dużo mówić, jestem fanatykiem make'a. Pół repozytorium zawalone Makefile'ami najgorsze. Średnio raz w miesiącu ktoś wdepnie w moje wildcardy czy dynamiczne targety i trzeba wyciągać z manuala, bo mają .PHONY na końcu... 
A tak na poważnie, to make jest nadal świetnym narzędziem, a połączenie go z testami typu "golden master" (porównujących stan ustalony z obecnym) jest sprytne i bardzo użyteczne. Do czego? Zarówno do "normalnego" kodu jak i do programowalnej infrastruktury, np. Terraforma czy Jsonneta. 

### Paweł poleca: Zapraszam do motora

[A Generic Sidecar Injector for Kubernetes](https://engineering.salesforce.com/a-generic-sidecar-injector-for-kubernetes-c05eede1f6bb)

Salesforce pokazuje prosty framework do generowania sidecarów w Kubernetesie. Pod linkiem możecie przeczytać krótki wstęp czym są sidacary, oraz jak często taka firma jak Salesforce korzysta z takiego wzorca. Ale głównym celem jest "sprzedanie" frameworka, który zbudowali. Za kilka linii w yamlu dostaniecie jeden kontener gratis do każdego poda, jest to kusząca oferta.

### Jarek poleca: Lewicowe i prawicowe spojrzenia na testowanie kodu

[DevSecOps: Shifting Left and Shifting Right](https://hackernoon.com/devsecops-shifting-left-and-shifting-right-m5do32zj)

Kolejna dawka haseł ze słownika DevOps, tym razem "przesunięcia" testów.  
Przesunięcie w lewo (shift left) to pogląd, wedle którego należy dążyć do przeprowadzania jak największej liczby testów jak najwcześniej, bo im bliżej powstania tym ich usunięcie jest "tańsze".
Natomiast przesunięcie w prawo (shift right) podpowiada, żeby jak najszybciej wydawać taki kod, jaki mamy i weryfikować jego jakość już na docelowym środowisku (np. poprzez [automatyczną weryfikację wdrożeń kanarkowych](https://netflixtechblog.com/automated-canary-analysis-at-netflix-with-kayenta-3260bc7acc69)).
Które podejście jest lepsze?
Nie chodzi o to, żeby wybrać jedno z nich za jedyne prawidłowe, ale warto poznać oba koncepty, żeby móc wykorzystać ich wady i zalety podejmując decyzje na temat procesu testowania.
Najlepiej będzie połączyć obie filozofie, dobierając odpowiednie rodzaje testów do każdej fazy - wtedy korzyści będą podwójne.
Uwaga: w artykule znajduje się drobny marketing.

