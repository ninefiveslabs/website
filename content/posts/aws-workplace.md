---
draft: false
date: 2020-05-04T01:16:13+02:00
title: "Przygotowanie środowiska do pracy z AWS"
slug: "aws-workplace"
series: ["aws-bootstrap"]
tags: [aws]
---

_To pierwszy wpis z serii o fundamentach AWS. Kolejne będą pojawiały się w [tym spisie]({{< ref "/series/aws-bootstrap" >}})._

Za każdym razem gdy siada się do chmurowego projektu na nowym koncie AWS, chce się wypróbować nowe lub nieznane usługi albo pouczyć się do certyfikatu, pojawia się problem przygotowania i skonfigurowania środowiska pracy.

Chmura Google rozwiązuje ten problem (przynajmniej częściowo) poprzez swój [Cloud Shell](https://cloud.google.com/shell) - darmowy terminal w przeglądarce, w którym można skorzystać z zawsze aktualnych narzędzi konsolowych do pracy z GCP. W przypadku AWS nie jest już tak prosto i - choć da się stworzyć podobne narzędzie samemu (np. instancja EC2 z przypisaną odpowiednią rolą IAM) - to będzie to wymagało dodatkowej pracy, a użyte zasoby będą płatne. Dlatego do celów szkoleniowo-testowych zdecydowałem się na pracę z AWS z konsoli laptopa.

W trakcie zbierania informacji na temat tego jak skonfigurować swoje środowisko robocze do AWS trafiłem na kilka rzeczy, które chciałbym zrobić inaczej niż w podstawowej dokumentacji. Po pierwsze, domyślny sposób przechowywania kluczy dostępowych do AWS to niezaszyfrowany plik w katalogu domowym: komenda `aws configure` zapisuje je do `~/.aws/credentials`. Nawet jeżeli korzystamy z wieloetapowego uwierzytelniania (co jest rekomendowaną metodą dla użytkowników-ludzi), trzymanie niezabezpieczonych kluczy na dysku to nienajlepszy pomysł. Po drugie, oficjalna metoda instalacji narzędzia `aws-cli` to ściągnięcie paczki zip i uruchomienie skryptu instalacyjnego przy użyciu sudo[^1] - skrypt kopiuje sporą liczbę plików do globalnych folderów systemowych. Po trzecie, wieloetapowe uwierzytelnianie kodem OTP w konsoli wymaga wykonania dość długiej komendy i żonglerki kluczami dostępowymi, co warto trochę zautomatyzować. Po zlokalizowaniu problemów do pokonania i przeczytaniu kilku artykułów na ten temat (spis znajdziesz na końcu artykułu) postanowiłem zebrać najlepsze cechy znalezionych rozwiązań i napisać własny prosty skrypt, który będzie moim codziennym kompanem w pracy z chmurą Amazona.

## Dla niecierpliwych

Klonujemy [repozytorium git ze skryptami](https://github.com/ninefiveslabs/aws-workplace):
```
git clone https://github.com/ninefiveslabs/aws-workplace.git
cd aws-workplace
```

Uruchamiamy skrypt `init.sh`, który zaszyfruje klucze:
```
./init.sh
```
Skrypt poprosi kolejno o: access key ID, secret access key, identyfikator urządzenia MFA (do znalezienia na podanej stronie, lub do założenia w [ten sposób](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html)), domyślny region i hasło do zaszyfrowania kluczy w pliku.
```
Enter your AWS_ACCESS_KEY_ID:
Enter your AWS_SECRET_ACCESS_KEY:
Enter your MFA device ARN (see: https://console.aws.amazon.com/iam/home#/security_credentials):
Enter default region [us-east-1]:
Now comes the GPG encryption password prompt:
Enter passphrase:
```

Teraz za każdym razem, gdy będziemy chcieli przygotować konsolę do pracy z AWS, należy wczytać plik `env.sh`, który poprosi o podanie hasła do rozszyfrowania kluczy dostępowych oraz aktualnego kodu MFA:

```
. env.sh
```

```
Updating aws-cli...
gpg: AES256 encrypted data
Enter passphrase:
Enter current MFA token:
Session expires at: 2020-05-04T00:00:00+00:00
```

Po zakończeniu skryptu `env.sh` nasza konsola zawiera odpowiednie zmienne środowiskowe z danymi do uwierzytelniania (klucze sesji), a w `PATH` znajduje się gotowa do użycia komenda `aws` w najnowszej wersji.

Aby wyczyścić wszystkie zmienne środowiskowe i dopiski do `PATH` i `PS1` można wczytać skrypt `clear-env.sh`:
```
. clear-env.sh
```

## Opis rozwiązania

Celem powstania skryptu było zapewnienie większego bezpieczeństwa niż przechowywanie kluczy dostępowych w nieszyfrowanym pliku, łatwość obsługi ze wsparciem dla MFA i dodatkowo wsparcie w instalacji i aktualizacji narzędzi konsolowych.

Klucze są szyfrowane przy użyciu `gpg` w trybie symetrycznym. Przy każdej próbie logowania wykonywane są następujące kroki:

1. próba instalacji/aktualizacji narzędzia `aws-cli`
1. modyfikacja `PATH`, tak żeby dodatkowe narzędzia AWS były dostępne w konsoli
1. wczytanie hasła w terminalu
1. odszyfrowanie kluczy dostępowych i załadowanie ich do zmiennych środowiskowych
1. wczytanie kodu MFA w terminalu
1. utworzenie sesji w usłudze Session Token Service
1. podmiana kluczy w zmiennych środowiskowych na "sesyjne" (po uwierzytelnianiu MFA otrzymujemy nowy identyfikator klucza, klucz tajny oraz klucz sesji, którymi należy się posługiwać w zapytaniach do AWS)
1. dopisanie do `PS1` insformacji, że znajdujemy się w środowisku gotowym do pracy z AWS

Skrypty zostały napisane zgodnie z następującymi regułami:
- niezapisywanie danych tajnych na dysku (np. w plikach tymczasowych)
- nieprzekazywanie danych tajnych w argumentach poleceń (np. `echo` - listy argumentów są dostępne dla wszystkich użytkowników systemu, zmienne środowiskowe tylko dla właściciela procesu)
- nieużywanie `eval` i `source` do wczytywania plików konfiguracyjnych (zamiast tego `read`)
- zgodność z testami [shellcheck](https://github.com/koalaman/shellcheck)
- pobieranie narzędzi do lokalnych katalogów (zamiast globalnej instalacji systemowej z użyciem `sudo`).

W ten sposób stworzyłem sobie wygodne poletko do eksperymentowania i prototypowania z AWS. Korzystam z tych narzędzi w założeniu, że efekty mojej pracy finalnie trafiają do narzędzia CI/CD i wdrożenie odbywa się automatycznie, na podstawie kodu trzymanego w repozytorium. Jednak możliwość skorzystania z linii poleceń i ręcznego wywołania API albo uruchomienia Terraforma doskonale przyspiesza tworzenie kodu, zanim trafi on ostatecznie do jakiegoś zadania CI/CD. Jest to też sposób na debugowanie istniejących aplikacji.

## Źródła inspiracji

Te materiały pomogły mi stworzyć końcowe rozwiązanie:
- https://hackernoon.com/aws-credentials-stored-safer-m5673wd3
- https://dev.to/michrodz/use-mfa-on-the-cli-and-execute-awscli-commands-securely-3i8c
- https://github.com/realestate-com-au/credulous

## Seria fundamenty AWS

W następnym wpisie z tej serii opiszę jak zacząć pracę z [AWS Cloud Development Kit](https://aws.amazon.com/cdk/) - frameworkiem skryptowym do tworzenia zasobów na AWS. A gdy już opanujemy tworzenie infrastruktury i wdrażanie aplikacji przy użyciu CDK, w trzecim wpisie nauczymy się jak sprzątać je narzędziem [aws-nuke](https://github.com/rebuy-de/aws-nuke). Nowe wpisy będą pojawiać się na [stronie serii]({{< ref "/series/aws-bootstrap" >}})

[^1]: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html#cliv2-linux-install