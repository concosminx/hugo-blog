---
date: '2026-09-04T10:00:00+03:00'
draft: false
title: 'Cheatsheet pentru linia de comandă Windows: fișiere, foldere și rețelistică'
tags: ["windows", "cheatsheet"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "O referință practică pentru Command Prompt-ul din Windows — navigarea prin sistemul de fișiere, gestionarea fișierelor și diagnosticarea problemelor de rețea cu ipconfig, ping, tracert, netstat și celelalte."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

PowerShell este de ani buni shell-ul implicit pe Windows, dar `cmd.exe` refuză să dispară — și pe bună dreptate. Pornește instantaneu, există pe orice mașină Windows livrată vreodată, funcționează în Safe Mode și în consolele de recuperare, iar jumătate dintre diagnosticele de rețea de care vei avea nevoie sunt la o comandă scurtă distanță. Când te sună cineva pentru că „nu merge internetul", `Win`+`R` → `cmd` rămâne cea mai rapidă cale către un răspuns.

Acesta este setul de comenzi care merită ținut minte: mai întâi bazele legate de sistemul de fișiere, apoi trusa de rețelistică ce face Command Prompt-ul cu adevărat util.

## Unde ești și ce se află în jur

Un filepath este pur și simplu poziția ta în sistemul de fișiere. `C:` este unitatea C, `C:\Users\Me\Documents` este un folder din interiorul ei, iar `C:\Users\Me\Documents\notes.txt` este un fișier din acel folder. Tot ce urmează presupune că știi în care dintre ele te afli.

`dir` afișează conținutul folderului curent:

```bat
dir
dir myfolder
```

Două opțiuni transformă `dir` dintr-o listare într-un instrument de căutare. `/a` include fișierele ascunse și de sistem, pe care listarea simplă le omite discret — de obicei acesta este motivul pentru care un folder „nu este gol" atunci când încerci să îl ștergi. `/s` intră recursiv în subfoldere, așa că `dir /s /b *.log` parcurge un arbore întreg și afișează căile brute, fără antete.

`cd` (sau aliasul său mai lung, `chdir`) schimbă directorul, iar fără argument afișează pur și simplu unde te afli:

```bat
cd C:\Users\Me\Documents
cd ..
cd
```

`cd ..` urcă un nivel. O capcană care păcălește pe toată lumea: `cd D:\some\path` dintr-un prompt aflat pe `C:` *nu* te va muta acolo. Fiecare unitate își păstrează propriul director curent, așa că schimbi unitatea tastând litera ei singură sau folosind `cd /d`, care face ambele lucruri deodată:

```bat
D:
cd /d D:\some\path
```

Pentru o privire de ansamblu asupra unui arbore întreg, nu doar asupra unui nivel, `tree` îl desenează în ASCII. Adaugă `/f` ca să incluzi și fișierele, nu doar folderele:

```bat
tree C:\Projects /f
```

## Creare, mutare și ștergere

`md` creează un folder; `mkdir` este aceeași comandă sub alt nume.

```bat
md new-folder
```

Ștergerea este locul unde fișele de referință greșesc suficient de des încât să merite spus răspicat: comanda din Command Prompt este `rd`, sau echivalent `rmdir`. **Nu există `rm` în `cmd.exe`** — `rm` funcționează doar în PowerShell, unde este un alias pentru `Remove-Item`. Tastat într-un Command Prompt îți dă „not recognized as an internal or external command".

```bat
rd empty-folder
rd /s /q folder-with-stuff-in-it
```

`rd` simplu refuză să șteargă orice mai conține fișiere. `/s` șterge directorul și tot ce se află sub el, iar `/q` suprimă cererea de confirmare. `/q` are efect doar atunci când este prezent și `/s` — iar combinația șterge un arbore întreg fără să întrebe, așa că citește calea de două ori înainte de a apăsa Enter.

Pentru fișiere, nu foldere, `del` acceptă unul sau mai multe nume și lucrează cu wildcard-uri:

```bat
del report.txt
del *.tmp
```

`copy` duplică un fișier, `move` îl relocalizează, iar `ren` (forma lungă `rename`) îi schimbă numele pe loc:

```bat
copy C:\data\report.txt D:\backup\
move folder1\file.txt folder2\
ren oldname.txt newname.txt
```

`copy` este potrivit pentru câteva fișiere. Pentru orice seamănă cu un backup real — arbori întregi, transferuri care se pot relua, reîncercări pentru fișierele blocate — folosește `robocopy`, care vine cu Windows și este construit exact pentru asta.

## Citirea fișierelor și controlul ecranului

`type` afișează un fișier text în consolă, cea mai rapidă cale de a verifica un fișier de configurare sau un log scurt:

```bat
type C:\Windows\System32\drivers\etc\hosts
```

`fc` compară două fișiere și afișează diferențele, ceea ce este mai util decât pare atunci când încerci să îți dai seama care dintre două fișiere de configurare aproape identice este cel stricat:

```bat
fc config.old config.new
```

`echo` afișează un mesaj, iar `cls` curăță ecranul. `exit` închide Command Prompt-ul sau termină scriptul batch curent.

```bat
echo Backup finished
cls
```

## Cum obții ajutor

Două forme, care fac lucruri diferite. `help` singur listează comenzile interne, iar `help <comandă>` explică una dintre ele:

```bat
help
help rd
```

Dar `help` cunoaște doar comenzile integrate în `cmd.exe`. Pentru toate celelalte — `ping`, `netstat`, `ipconfig` — adaugă `/?` la comanda respectivă:

```bat
ping /?
netstat /?
```

Când eziți, `/?` este varianta de ales; funcționează pentru ambele categorii.

## Propria configurație de rețea

`ipconfig` este punctul de plecare pentru orice întrebare legată de rețea. Rulat simplu, afișează adresa IP, masca de subrețea și gateway-ul implicit pentru fiecare adaptor. `/all` este versiunea pe care o vrei de fapt — adaugă adresa MAC, serverul DHCP, durata lease-ului și serverele DNS configurate:

```bat
ipconfig
ipconfig /all
```

Opțiunile legate de DNS sunt cele care chiar rezolvă probleme, nu doar le descriu:

```bat
ipconfig /displaydns
ipconfig /flushdns
ipconfig /registerdns
```

`/displaydns` arată ce a memorat resolver-ul în cache, `/flushdns` golește acel cache, iar `/registerdns` reînnoiește lease-ul DHCP și reînregistrează numele mașinii în DNS. `/flushdns` este răspunsul standard la „am schimbat înregistrarea DNS acum o oră și mașina asta tot merge la serverul vechi". `/registerdns` are nevoie de un prompt cu drepturi de administrator.

## Este accesibil?

`ping` trimite cereri ICMP echo și raportează ce se întoarce. Implicit trimite patru și se oprește.

```bat
ping 192.168.1.144
ping google.com
```

Opțiunile care merită știute:

| Opțiune | Ce face |
| --- | --- |
| `-t` | Trimite continuu până îl oprești cu `Ctrl`+`C`. `Ctrl`+`Break` afișează statisticile fără să iasă. |
| `-n <număr>` | Trimite exact atâtea cereri, în loc de patru. |
| `-l <dimensiune>` | Setează dimensiunea în octeți a datelor din pachet. Implicit 32, maximum 65500. |
| `-f` | Setează flag-ul „don't fragment". Doar pentru IPv4. |
| `-w <ms>` | Cât se așteaptă fiecare răspuns. Implicit 4000 ms. |
| `-4` / `-6` | Forțează IPv4 sau IPv6. Necesar doar când dai un hostname, nu o adresă. |
| `-a` | Rezolvă invers adresa țintă la un hostname. |

Observă ce face de fapt `-l`: setează dimensiunea datelor din pachet, nu îți permite să editezi conținutul pachetului. Combinat cu `-f` devine un test de path MTU — trimiți un pachet mare, care nu poate fi fragmentat, și vezi dacă se plânge ceva pe traseu:

```bat
ping -f -l 1300 8.8.8.8
```

Dacă un router de pe traseu are un MTU mai mic decât pachetul trimis, nu îl poate fragmenta și nu îl poate transmite mai departe, așa că răspunde cu „Packet needs to be fragmented but DF set". Scazi dimensiunea până când ping-urile trec și ai găsit MTU-ul efectiv. Pentru că `-f` este o opțiune exclusiv IPv4, trucul nu funcționează peste IPv6.

Un ping eșuat este o dovadă mai slabă decât presupun majoritatea. Destule mașini — inclusiv Windows Firewall în configurația implicită — aruncă ICMP-ul din principiu, deși servesc traficul perfect. „Ping-ul nu merge" înseamnă „ICMP-ul nu se întoarce", nu „mașina este picată".

## Pe unde trece traficul, de fapt

`tracert` listează fiecare router de pe traseul dintre tine și destinație:

```bat
tracert google.com
tracert -d 8.8.8.8
```

`-d` sare peste rezolvarea DNS inversă pentru fiecare hop, ceea ce face ca rezultatul să apară mult mai repede când router-ele intermediare nu au înregistrări PTR. `-h <număr>` limitează numărul de hop-uri, util când te interesează doar primele și nu vrei să aștepți treizeci.

`pathping` este `tracert` plus măsurare susținută. Întâi mapează traseul, apoi dă ping repetat fiecărui hop și raportează latența și pierderea de pachete per hop:

```bat
pathping -n google.com
```

Pregătește-te să aștepți. Implicit trimite 100 de interogări către fiecare hop, la intervale de 250 ms, ceea ce înseamnă aproximativ 25 de secunde per hop — un traseu de zece hop-uri durează cam patru minute. `-q` reduce numărul de interogări, iar `-p` scurtează intervalul, dacă vrei un răspuns mai devreme. Câștigul este că distinge între un router care tratează cu prioritate scăzută ICMP-ul adresat lui însuși și o legătură care chiar pierde traficul transmis mai departe — ceva ce un `tracert` simplu nu poate face.

## Cu ce vorbește mașina asta?

`netstat` arată conexiunile și porturile pe care se ascultă. Aproape nimeni nu îl rulează gol; combinația utilă este:

```bat
netstat -ano
```

`-a` include porturile în ascultare, nu doar conexiunile stabilite, `-n` afișează adresele și porturile numeric în loc să încerce să rezolve nume, iar `-o` adaugă ID-ul procesului deținător, ca să poți lega un port de ceva din Task Manager. `-n` merită adăugat fie și doar pentru viteză — rezolvarea numelor pe o listă lungă de conexiuni este lentă.

Ca să treci direct de la un port la numele executabilului, înlocuiește `-o` cu `-b`:

```bat
netstat -ab
```

`-b` are nevoie de un prompt cu drepturi de administrator și eșuează fără el, iar în plus este vizibil mai lent decât `-o`. Un proces gazdă precum `svchost.exe` afișează lanțul de componente între paranteze drepte, nu un singur nume.

Două lucruri pe care ți le arată rezultatul. Un proces care ascultă pe `0.0.0.0` acceptă conexiuni pe orice interfață, în timp ce unul legat de `127.0.0.1` este accesibil doar de pe mașina însăși — distincția asta explică o grămadă de sesizări de tipul „local merge, de pe laptopul meu nu". Și când un serviciu refuză să pornească cu „address already in use", `netstat -ano` plus PID-ul îți spune exact cine a ajuns acolo primul.

Pentru contoare pe protocol, nu pentru conexiuni individuale, `-s` afișează statistici per protocol — IPv4, IPv6, TCP, UDP și ICMP:

```bat
netstat -s -p tcp
```

`netstat -r` afișează tabela de rutare și este identic cu `route print`.

## Rezolvarea numelor

`nslookup` interoghează DNS-ul direct, ceea ce contează pentru că întreabă un server, nu raportează ce crede deja clientul:

```bat
nslookup www.google.com
```

Rulat fără argumente intră într-un mod interactiv în care poți schimba tipul înregistrării și interoga repetat. `set q=mx` pentru înregistrările de mail, `set q=cname` pentru alias-uri, `set q=ns` pentru serverele de nume autoritative:

```bat
nslookup
> set q=mx
> example.com
> exit
```

Îl poți îndrepta și către un server anume, adăugându-l ca al doilea argument — `nslookup example.com 8.8.8.8` întreabă resolver-ul Google în loc de al tău. Compararea răspunsului serverului tău DNS cu cel al unuia public este cea mai rapidă cale de a confirma o înregistrare învechită.

## Cache-ul ARP

ARP mapează adresele IP la adrese MAC în segmentul local. Cache-ul este ceea ce crede mașina ta în acest moment despre vecinii ei:

```bat
arp -a
```

Intrările sunt fie dinamice (învățate de la un vecin și expirate în timp), fie statice (adăugate manual sau de sistem). `FF-FF-FF-FF-FF-FF` este adresa de broadcast, nu o mașină reală.

Poți șterge o intrare sau adăuga una:

```bat
arp -d 192.168.1.1
arp -s 192.168.1.1 00-AA-22-BB-33-CC
```

Ambele au nevoie de un prompt cu drepturi de administrator. O limitare a lui `arp -s` merită știută înainte să te bazezi pe el: intrarea este eliminată de fiecare dată când stiva TCP/IP este oprită și repornită, deci nu supraviețuiește unui restart. Pentru o mapare statică persistentă, folosește `netsh interface ipv4 add neighbors "Ethernet" 192.168.1.1 00-aa-22-bb-33-cc` sau echivalentul din PowerShell, `New-NetNeighbor` — ambele stochează intrarea persistent, implicit.

Ștergerea unei intrări este, în practică, mișcarea mai frecventă — după înlocuirea unui router sau după un failover, o intrare ARP învechită care indică vechiul MAC va înghiți traficul până expiră, iar `arp -d` scutește așteptarea.

## Tabela de rutare

`route print` afișează tabela de rutare a mașinii:

```bat
route print
```

Intrarea pe care o cauți este ruta implicită: destinație `0.0.0.0`, mască `0.0.0.0`, îndreptată către gateway-ul tău. Aceasta este regula „tot restul merge pe aici", iar absența ei explică pierderea totală a conectivității externe în timp ce rețeaua locală continuă să funcționeze. Un client de VPN care s-a închis prost este o cauză frecventă pentru o rută rămasă în urmă, care trimite traficul într-un tunel ce nu mai există.

Adăugarea unei rute are nevoie de o destinație, o mască și un gateway — un simplu `route add 192.168.1.1` nu va face nimic util:

```bat
route add 10.41.0.0 mask 255.255.0.0 10.27.0.1
route -p add 10.41.0.0 mask 255.255.0.0 10.27.0.1
route delete 10.41.0.0
```

Fără `-p`, ruta trăiește doar până la repornirea stivei TCP/IP. `-p` o face persistentă, scriind-o în registry. Toate trei comenzile au nevoie de un prompt cu drepturi de administrator.

## Testarea unui singur port

Ping-ul îți spune dacă o mașină răspunde la ICMP. Nu îți spune nimic despre faptul că serviciul care te interesează ascultă sau nu. Pentru asta, deschide o conexiune TCP către portul respectiv:

```bat
telnet mail.example.com 25
telnet example.com 80
```

Un ecran gol sau un banner al serviciului înseamnă că portul este deschis și accesibil. Un „Could not open connection to the host" imediat înseamnă că ceva — serviciul, un firewall sau un echipament de rețea de pe traseu — l-a refuzat sau l-a aruncat. Portul implicit al telnet-ului este 23, dar ca test de conectivitate exact portul pe care îl indici tu este esențial: 25 pentru SMTP, 80 pentru HTTP, 443 pentru HTTPS, 3389 pentru RDP.

O problemă: clientul Telnet nu mai este instalat implicit din Windows Vista încoace. Activează-l o singură dată, dintr-un prompt cu drepturi de administrator:

```bat
dism /online /Enable-Feature /FeatureName:TelnetClient
```

Dacă preferi să nu instalezi nimic, `Test-NetConnection -ComputerName example.com -Port 443` din PowerShell face același test și raportează rezultatul, în loc să te lase într-o sesiune brută.

## NetBIOS, dacă încă îți trebuie

`nbtstat` acoperă NetBIOS peste TCP/IP — rezolvarea numelor din epoca dinaintea generalizării DNS-ului în rețelele Windows. Într-o rețea modernă vei avea rareori nevoie de el, dar share-urile de fișiere vechi și aplicațiile de business învechite încă se sprijină pe el.

```bat
nbtstat -n
nbtstat -c
nbtstat -A 10.0.0.99
```

`-n` arată tabela de nume NetBIOS a mașinii locale, `-c` arată cache-ul de nume — numele pe care le-a rezolvat și adresele către care le-a rezolvat. Sesiunile sunt o opțiune diferită: `-s` listează sesiunile NetBIOS client și server cu nume, iar `-S` le listează pe aceleași doar cu adrese IP. Confuzia dintre `-c` și `-s` se face ușor și îți dă cu totul altă tabelă. Pentru interogări la distanță, `-a` cu literă mică primește un nume NetBIOS, iar `-A` cu literă mare primește o adresă IP — `nbtstat` este una dintre puținele comenzi Windows unde majusculele chiar contează.

## Echivalentele din PowerShell

Nimic din toate acestea nu dispare, dar PowerShell are înlocuitori de primă mână pentru majoritatea, iar aceștia întorc obiecte pe care le poți filtra și sorta, nu text pe care trebuie să îl parsezi:

| Command Prompt | PowerShell |
| --- | --- |
| `ipconfig /all` | `Get-NetIPConfiguration -Detailed` |
| `ipconfig /flushdns` | `Clear-DnsClientCache` |
| `ping` | `Test-Connection` |
| `telnet host port` | `Test-NetConnection -ComputerName host -Port port` |
| `tracert` | `Test-NetConnection -TraceRoute` |
| `nslookup` | `Resolve-DnsName` |
| `netstat -ano` | `Get-NetTCPConnection` |
| `arp -a` | `Get-NetNeighbor` |
| `route print` | `Get-NetRoute` |

Comenzile clasice rămân alegerea corectă când ești pe o mașină necunoscută, într-un mediu de recuperare sau când citești scriptul altcuiva. Învață-le pe ambele — te costă cam o după-amiază împreună, iar una dintre ele este întotdeauna disponibilă.

Microsoft documentează fiecare comandă din acest articol în [referința Windows Commands](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands), locul unde să mergi când `/?` este prea concis.
