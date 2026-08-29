---
date: '2026-08-29T10:00:00+03:00'
draft: false
title: 'Comenzi cURL — Cheatsheet'
tags: ["cheatsheet", "cli"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Opțiunile curl care merită reținute — trimiterea datelor, inspectarea header-elor, gestionarea cookie-urilor și a timeout-urilor, plus folosirea codurilor de ieșire în scripturi."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

`curl` are peste două sute de opțiuni, iar tu vei folosi vreo douăzeci. Acestea sunt cele douăzeci — opțiunile care apar când depanezi un API, scrii un script de verificare a stării unui serviciu sau încerci să înțelegi de ce o cerere se comportă diferit față de browser.

## Salvarea rezultatului

Implicit, curl scrie corpul răspunsului la stdout. Două opțiuni îl redirecționează, iar diferența dintre ele este de unde vine numele fișierului:

```bash
curl -o page.html https://example.com
curl -O https://example.com/archive.tar.gz
```

`-o` folosește numele pe care i-l dai tu. `-O` refolosește numele de la distanță — comod pentru descărcări, dar preia numele din URL, așa că un URL care se termină cu un segment de cale și nu cu un nume de fișier îți dă ceva nefolositor.

Redirecturile nu sunt urmate decât dacă ceri asta explicit:

```bash
curl -L https://example.com
```

Fără `-L`, un 301 returnează chiar răspunsul de redirect, nu pagina. Acesta este cel mai frecvent motiv pentru care o cerere curl „nu întoarce nimic" deși browserul încarcă pagina fără probleme.

## Trimiterea cererilor

Metoda este implicit GET, sau POST atunci când atașezi date. Seteaz-o explicit cu `-X`:

```bash
curl -X DELETE https://api.example.com/items/42
```

Datele se trimit cu `-d`:

```bash
curl -d 'name=value&other=thing' https://api.example.com/submit
curl -d @payload.json https://api.example.com/submit
```

Prefixul `@` citește corpul cererii dintr-un fișier. Reține că `-d` procesează acel fișier — elimină newline-urile și carriage return-urile — ceea ce e în regulă pentru JSON, dar corupe orice conținut binar. Când octeții trebuie să ajungă exact așa cum sunt, folosește forma binară:

```bash
curl --data-binary @image.png https://api.example.com/upload
```

Header-ele personalizate se pot repeta, câte o opțiune pentru fiecare header:

```bash
curl -H 'Content-Type: application/json' -H 'Authorization: Bearer token' \
     -d @payload.json https://api.example.com/submit
```

User agent-ul are propria scurtătură, pentru că suprascrierea lui este suficient de frecventă cât să merite una:

```bash
curl -A 'Mozilla/5.0' https://example.com
```

## Inspectarea schimbului

Trei niveluri de detaliu, în funcție de cât de mult ai nevoie să vezi.

`-i` include header-ele răspunsului deasupra corpului:

```bash
curl -i https://example.com
```

`-I` trimite o cerere HEAD și afișează doar header-ele, fără să transfere deloc corpul — alegerea potrivită pentru a verifica un cod de stare sau ținta unui redirect:

```bash
curl -I https://example.com
```

`-v` arată întreaga conversație, inclusiv cererea pe care curl a trimis-o efectiv:

```bash
curl -v https://example.com
```

În ieșirea verbose, `>` marchează datele trimise de curl, iar `<` datele primite. Această distincție este ceea ce face `-v` util: majoritatea bug-urilor de tipul „serverul greșește" se dovedesc a fi un header pe care nu l-ai trimis.

Direcția opusă, pentru scripturi:

```bash
curl -s https://example.com
curl -sS https://example.com
```

`-s` ascunde indicatorul de progres și mesajele de eroare. Asta ascunde însă și erorile reale, așa că folosește-l împreună cu `-S`, care readuce erorile. `-sS` este combinația pe care o vrei într-un cron job — tăcut când funcționează, gălăgios când nu.

## Cookie-uri

`-b` trimite cookie-uri, iar `-c` le stochează. Folosite împreună îți oferă o sesiune între mai multe rulări:

```bash
curl -c jar.txt -d 'user=me&pass=secret' https://example.com/login
curl -b jar.txt https://example.com/account
```

`-b` acceptă și cookie-uri scrise direct în linia de comandă, ceea ce e mai rapid când ai nevoie doar de unul sau două:

```bash
curl -b 'session=abc123; theme=dark' https://example.com
```

## Autentificare și proxy

Autentificarea HTTP primește o pereche separată prin două puncte:

```bash
curl -u username:password https://example.com
```

Dacă omiți parola (`-u username`), curl o va cere interactiv, ceea ce ține credențialul în afara istoricului shell-ului și în afara listei de procese — merită tasta în plus pe o mașină partajată.

Rutarea printr-un proxy:

```bash
curl -x 'http://proxy:2080' https://example.com
```

## Timeout-uri și reîncercări

Cererile fără limită de timp sunt motivul pentru care un script rămâne blocat la infinit. Două limite diferite, care nu sunt interschimbabile:

```bash
curl --connect-timeout 5 -m 30 https://example.com
```

`--connect-timeout` limitează doar timpul petrecut pentru stabilirea conexiunii; `-m` limitează întreaga operațiune, inclusiv transferul. O descărcare mare are nevoie de un `-m` generos, dar beneficiază totuși de un `--connect-timeout` strâns, așa că de obicei e corect să le setezi pe amândouă.

Pentru erori tranzitorii, curl poate reîncerca singur:

```bash
curl --retry 3 --retry-delay 2 https://example.com
```

## Compresie

Cere serverului să comprime răspunsul:

```bash
curl --compressed https://example.com
```

curl anunță formatele de codare cu care a fost compilat — de regulă gzip, deflate, br și zstd — și decomprimă răspunsul transparent, așa că la stdout ajunge același conținut în ambele cazuri. Opțiunea este `--compressed`, nu `--compression`; a doua nu există în curl și va eșua din start.

## TLS

Eșecurile de verificare a certificatului opresc cererea. Pentru a continua oricum:

```bash
curl -k https://self-signed.example.com
```

`-k` (forma lungă `--insecure`) este pentru testarea pe servere de staging și certificate self-signed. Dezactivează complet verificarea, ceea ce înseamnă că dezactivează și protecția pe care te bazai, așa că nu are ce căuta în ceva ce rulează nesupravegheat.

## Scripting cu curl

Două funcționalități transformă curl dintr-un instrument interactiv într-o componentă utilizabilă în scripturi.

Prima este `-w`, care afișează un șir de format ales de tine după un transfer reușit:

```bash
curl -s -w '%{remote_ip} %{time_total} %{http_code}\n' -o /dev/null https://example.com
```

Trimiterea corpului către `/dev/null` și afișarea doar a metricilor îți oferă o verificare de stare pe un singur rând. Variabile utile:

| Variabilă | Semnificație |
| --- | --- |
| `%{http_code}` | Codul de stare al răspunsului |
| `%{time_total}` | Timpul total de transfer, în secunde |
| `%{time_connect}` | Timpul până la stabilirea conexiunii |
| `%{time_starttransfer}` | Timpul până la primul octet |
| `%{remote_ip}` | IP-ul la care s-a conectat efectiv curl |
| `%{size_download}` | Octeți descărcați |
| `%{json}` | Toate variabilele disponibile, ca obiect JSON |

A doua sunt codurile de ieșire. curl întoarce 0 pentru un *transfer finalizat*, ceea ce include și un 404 sau un 500 — cererea a funcționat, doar că serverul a răspuns negativ. Pentru ca erorile HTTP să facă și comanda să eșueze:

```bash
curl -f https://example.com/missing
```

`-f` (`--fail`) suprimă corpul erorii și întoarce un cod de ieșire diferit de zero la erorile HTTP, exact ce îți trebuie când următoarea linie din script depinde de succes.

Codurile de ieșire pe care merită să le recunoști:

| Cod | Semnificație |
| --- | --- |
| 6 | Nu a putut rezolva host-ul |
| 7 | Nu s-a putut conecta la host sau proxy |
| 28 | Operațiunea a expirat |
| 55 | Eroare la trimiterea datelor |
| 56 | Eroare la primirea datelor |

Distincția dintre 6 și 7 este cea care îți economisește timp: 6 înseamnă DNS, 7 înseamnă conexiunea în sine. Să pornești `curl -v` înainte de a ști pe care dintre cele două o ai este efort irosit.

## O notă despre Windows

`curl.exe` vine inclus în Windows 10 și 11, dar în **Windows PowerShell 5.1** numele `curl` este un alias pentru `Invoke-WebRequest`, o comandă diferită, cu argumente complet diferite. Din acest motiv, o linie de comandă corectă eșuează cu erori derutante legate de parametri. Apelează executabilul cu numele complet:

```powershell
curl.exe -I https://example.com
```

PowerShell 7 a eliminat alias-ul, așa că acolo `curl` este comanda reală. În cmd.exe, Git Bash și WSL numele a trimis dintotdeauna către curl.

## Resurse suplimentare

`man curl` este lung, dar merită parcurs măcar o dată. [Documentația oficială](https://curl.se/docs/) acoperă opțiunile în întregime, iar `curl --help all` listează toate opțiunile din versiunea pe care o ai efectiv instalată — cel mai rapid mod de a verifica dacă o opțiune despre care ai citit undeva există în build-ul tău.
