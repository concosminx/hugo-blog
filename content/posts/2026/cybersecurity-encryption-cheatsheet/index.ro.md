---
date: '2026-09-04T11:00:00+03:00'
draft: false
title: 'Securitate Cibernetică și Criptare: Algoritmi, Protocoale și Atacuri'
taguri: ["cheatsheet", "cryptography", "security"]
categorii: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Cifruri simetrice și asimetrice, funcții hash, TLS, autentificare și control al accesului, gestionarea cheilor și atacurile împotriva cărora există toate acestea."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

O hartă de lucru a criptografiei aplicate și a mecanismelor de securitate construite peste ea. Multe dintre algoritmii de mai jos sunt aici pentru că îi vei întâlni în cod vechi și în întrebări vechi de examen, nu pentru că ar trebui să îi folosești — așa că fiecare vine însoțit de statutul său actual, adică exact partea pe care tabelele de referință o greșesc de obicei.

## Criptare simetrică

Algoritmii simetrici folosesc *aceeași* cheie secretă pentru criptare și decriptare. Sunt rapizi, motiv pentru care transportă datele propriu-zise în aproape orice protocol, iar punctul lor slab este distribuția cheii: ambele părți au nevoie de ea, iar transmiterea ei în siguranță este o problemă separată — exact cea pe care o rezolvă criptografia asimetrică.

| Algoritm | Tip | Dimensiune bloc | Dimensiune cheie | Statut |
| --- | --- | --- | --- | --- |
| AES | Bloc | 128 biți | 128 / 192 / 256 biți | **Standardul actual** |
| DES | Bloc | 64 biți | 56 biți | Compromis — atac prin forță brută |
| 3DES | Bloc | 64 biți | 112 / 168 biți | Interzis din 2023 |
| Blowfish | Bloc | 64 biți | până la 448 biți | Depășit |
| Twofish | Bloc | 128 biți | până la 256 biți | Nespart, puțin folosit |
| Serpent | Bloc | 128 biți | până la 256 biți | Nespart, puțin folosit |
| MARS | Bloc | 128 biți | 128–448 biți | Nespart, puțin folosit |
| IDEA | Bloc | 64 biți | 128 biți | Depășit |
| CAST-128 | Bloc | 64 biți | până la 128 biți | Depășit |
| RC2 | Bloc | 64 biți | până la 1024 biți | Nesigur |
| RC5 | Bloc | variabil | până la 2040 biți | Rar folosit |
| RC6 | Bloc | 128 biți | până la 2040 biți | Rar folosit |
| RC4 | Flux | — | până la 2048 biți | **Interzis** |

**AES** este cel care contează. Este standardul actual, este implementat hardware în orice procesor modern și, dacă nu ai un motiv anume să faci altfel, el este răspunsul. Observă că dimensiunea cheii variază, dar cea a blocului nu: AES-256 procesează tot blocuri de 128 de biți.

**DES** a căzut în fața atacurilor prin forță brută acum câteva decenii — 56 de biți pur și simplu nu înseamnă suficient material de cheie. **3DES** a fost soluția de compromis, aplicând DES de trei ori cu două sau trei chei distincte. Este lent, iar NIST l-a marcat ca depreciat până în 2023 și l-a interzis pentru criptare după 31 decembrie 2023, prin SP 800-131A Rev. 2; decriptarea datelor vechi rămâne permisă. Dacă găsești 3DES protejând ceva în producție, acela este un finding.

Cifrurile cu bloc de 64 de biți merită o notă separată. **Blowfish**, **3DES**, **IDEA**, **CAST-128** și **RC2** folosesc toate blocuri de 64 de biți, iar asta singură este acum o slăbiciune: atacul Sweet32, bazat pe paradoxul zilei de naștere, recuperează text clar din conexiuni de lungă durată criptate cu orice cifru cu bloc de 64 de biți, fără să atingă cheia. Blowfish este rapid și a fost mult timp de încredere, dar chiar autorul lui recomandă de ani buni trecerea la AES sau Twofish.

**Twofish**, **Serpent** și **MARS** au fost finaliste AES alături de Rijndael, care a câștigat. Niciunul nu este spart; pur și simplu au pierdut, iar asta a însemnat lipsa accelerării hardware și lipsa unui ecosistem. MARS a fost propunerea IBM — numele nu este un acronim, indiferent ce extinderi ai putea vedea atașate lui.

**RC4** este negativul important. Era peste tot, era rapid și acum este interzis formal în TLS prin RFC 7465. Nu trebuie negociat în nicio versiune de TLS.

O clarificare care merită făcută, pentru că fișele de referință îl trec constant la categoria greșită: **Kerberos nu este un algoritm de criptare.** Este un protocol de autentificare în rețea care *folosește* criptografie simetrică pentru a-și proteja mesajele. Revin asupra lui mai jos.

## Criptare asimetrică

Criptografia asimetrică — cu chei publice — folosește două chei legate matematic. Cheia publică poate fi dată oricui și servește la criptare; cheia privată rămâne secretă și servește la decriptare. Folosește perechea în sens invers și obții semnături digitale, de unde vine și proprietatea de **non-repudiere**: doar deținătorul cheii private putea produce semnătura, deci nu o poate nega în mod credibil.

Compromisul față de criptografia simetrică este viteza. Operațiile asimetrice sunt cu ordine de mărime mai lente, așa că în practică sunt folosite pentru a autentifica părțile și a stabili de comun acord o cheie simetrică, iar cifrul simetric face munca grea. Această aranjare hibridă este chiar ceea ce înseamnă TLS.

- **RSA** — calul de povară pentru transportul cheilor, semnături și criptarea datelor în repaus, numit după Rivest, Shamir și Adleman. Încă solid la chei de 2048 de biți și peste; RSA pe 1024 de biți este ieșit din uz.
- **Diffie-Hellman** — o metodă de *schimb* de chei, nu un algoritm de criptare. Permite două părți să deducă un secret comun pe un canal public fără să îl transmită vreodată. Variantele efemere (DHE, ECDHE) sunt cele care dau TLS-ului modern forward secrecy.
- **ECC (Elliptic Curve Cryptography)** — aceleași operații construite pe curbe eliptice, ceea ce oferă securitate echivalentă la dimensiuni de cheie mult mai mici. O cheie ECC de 256 de biți este aproximativ comparabilă cu una RSA de 3072 de biți, motiv pentru care ECC domină deopotrivă pe dispozitivele cu resurse limitate și în TLS-ul modern.
- **DSA** — Digital Signature Algorithm, doar pentru semnături. ECDSA este versiunea pe curbe eliptice și cea pe care o vei întâlni efectiv; Ed25519 le-a înlocuit în mare măsură pe amândouă în sistemele noi.
- **ElGamal** — un sistem cu cheie publică utilizabil atât pentru semnături, cât și pentru criptare. Rămâne mai ales de interes academic și istoric, deși supraviețuiește în părți din OpenPGP.
- **PGP** — nu un algoritm, ci un instrument, care combină criptarea simetrică și pe cea asimetrică pentru stocare și mesagerie sigure. Vezi [articolul despre GPG](../../2025/gpg/) pentru partea practică.

### Post-cuantic

Fiecare algoritm din această secțiune se sprijină pe probleme pe care un calculator cuantic suficient de mare le-ar rezolva eficient. NIST a publicat primele standarde post-cuantice în august 2024: **ML-KEM** (FIPS 203) pentru încapsularea cheilor, **ML-DSA** (FIPS 204) și **SLH-DSA** (FIPS 205) pentru semnături. Schimbul hibrid de chei, care combină ECDHE cu ML-KEM, este deja implementat în browserele și bibliotecile TLS de larg consum. Criptografia simetrică și funcțiile hash sunt mult mai puțin afectate — răspunsul practic acolo înseamnă chei mai mari și amprente mai lungi, nu algoritmi noi.

## Funcții hash

O funcție hash primește date de orice lungime și produce o amprentă de lungime fixă. Este unidirecțională: nu poți recupera intrarea, iar găsirea a două intrări cu aceeași amprentă — o coliziune — ar trebui să fie imposibilă din punct de vedere computațional. Funcțiile hash stau la baza semnăturilor, a verificărilor de integritate și a stocării parolelor.

| Familie | Dimensiune amprentă | Statut |
| --- | --- | --- |
| SHA-2 (SHA-256, SHA-512) | 224–512 biți | **Standardul actual** |
| SHA-3 (Keccak) | 224–512 biți | Actual, design intern diferit |
| BLAKE2 / BLAKE3 | variabilă | Actual, foarte rapid |
| SHA-1 | 160 biți | Compromis — coliziuni demonstrate |
| RIPEMD-160 | 160 biți | Nespart, dar de nișă |
| Whirlpool | 512 biți | Nespart, dar de nișă |
| Tiger | 192 biți | Vechi |
| MD5 | 128 biți | Compromis |
| MD4, MD2 | 128 biți | Compromise |

**SHA-2** este alegerea implicită, cu SHA-256 ca cel mai folosit membru. **SHA-3** nu este un petic peste SHA-2, ci un design structural diferit, ales printr-o competiție deschisă și păstrat în rezervă pentru cazul în care SHA-2 ar ceda vreodată. **BLAKE2** și succesorul său BLAKE3 sunt mai rapide decât ambele și apar pe scară largă în afara contextelor impuse de standarde.

**SHA-1** s-a terminat. O coliziune practică a fost demonstrată în 2017, iar coliziunile cu prefix ales au urmat în 2020. NIST l-a retras formal în 2022 și cere eliminarea lui completă până la finalul lui 2030. **MD5** are coliziuni demonstrate din 2004 și este inutilizabil în orice scop de securitate — singura utilizare care mai poate fi apărată este ca sumă de control într-un context fără adversar.

Întreaga familie MD produce amprente de 128 de biți: MD2, MD4 și MD5 deopotrivă. Toate sunt compromise.

O omisiune deliberată: **niciuna dintre acestea nu are ce căuta lângă o bază de date cu parole.** Funcțiile hash de uz general sunt proiectate să fie rapide, ceea ce este exact invers față de ce vrei la parole. Folosește o funcție intenționat lentă și consumatoare de memorie — Argon2id, scrypt sau bcrypt.

## TLS, nu SSL

SSL este mort. Toate versiunile lui — SSL 2.0 și SSL 3.0 — sunt formal depreciate, SSL 3.0 prin RFC 7568 în 2015. Succesorul său este TLS, iar TLS 1.0 și 1.1 sunt și ele depreciate, prin RFC 8996 în 2021. Doar **TLS 1.2 și TLS 1.3** ar trebui activate undeva. Cuvântul „SSL" supraviețuiește în numele bibliotecilor, în chei de configurare și în titluri de post, dar protocolul de dedesubt este TLS.

TLS are aceeași structură în două părți pe care o avea și SSL: un **handshake** care autentifică serverul și stabilește un secret comun, și un **protocol de înregistrare** care transportă datele aplicației criptate cu acel secret.

Handshake-ul clasic decurge cam așa:

1. Clientul trimite un mesaj hello care listează versiunile de protocol și suitele de cifruri pe care le suportă.
2. Serverul răspunde cu versiunea și suita de cifruri pe care le-a ales.
3. Serverul își prezintă certificatul digital pentru a-și dovedi identitatea.
4. Clientul validează acel certificat față de magazinul său de încredere, iar cele două părți finalizează schimbul de chei.
5. Tot ce urmează este criptat cu cheia simetrică stabilită.

**TLS 1.3 comprimă semnificativ acest proces.** Se finalizează într-un singur round trip în loc de două, criptează certificatul în loc să îl trimită în clar și elimină orice suită de cifruri cu o slăbiciune cunoscută — fără RC4, fără 3DES, fără schimb de chei RSA static, fără renegociere. Suitele sale sunt AES-GCM, AES-CCM și ChaCha20-Poly1305, toate cu criptare autentificată. Forward secrecy este obligatoriu, ceea ce înseamnă că o cheie de server furată nu poate decripta traficul capturat ieri.

Dacă configurezi un server astăzi: doar TLS 1.2 și 1.3, schimb de chei ECDHE, cifruri AEAD, HSTS activat. Generatorul de configurații SSL de la Mozilla produce configurații solide pentru majoritatea serverelor web.

## Autentificare

Autentificarea răspunde la întrebarea „cine ești?" și este împărțită convențional în trei factori — ceva ce știi, ceva ce ai, ceva ce ești.

| Metodă | Factor | Observații |
| --- | --- | --- |
| Utilizator și parolă | Știi | Cea mai slabă singură; baza peste tot |
| Doi factori / MFA | Știi + ai | Parolă plus un cod dintr-o aplicație sau de pe un token |
| Biometrie | Ești | Amprentă, față; comodă, dar nerevocabilă |
| Smart card | Ai | Card fizic cu cip, citit de un terminal |
| Bazată pe token | Ai | Cheie USB sau token hardware care păstrează credențialele |
| Bazată pe certificat | Ai | Certificat digital emis de o autoritate de încredere |

Autentificarea cu mai mulți factori este controlul cu cea mai mare valoare din acest tabel, dar nu toți factorii secundari sunt egali: codurile prin SMS pot fi interceptate prin SIM swapping și atacuri asupra rețelei, codurile TOTP generate de aplicații pot fi phishate în timp real, iar doar cheile de securitate hardware care folosesc WebAuthn/FIDO2 rezistă cu adevărat la phishing, pentru că cheia verifică originea înainte să semneze ceva.

Rezerva legată de biometrie merită spusă direct: nu îți poți reemite o amprentă. Biometria funcționează bine ca gest local de deblocare pe un dispozitiv care păstrează o credențială reală și prost ca secret transmis prin rețea.

### Protocoale de autentificare

- **Kerberos** — un protocol de autentificare în rețea care folosește criptografie simetrică și o terță parte de încredere, Key Distribution Center. Emite tichete cu durată limitată în loc să plimbe parole, ceea ce îl protejează împotriva atacurilor de tip replay, a interceptării și a atacurilor machine-in-the-middle. Este coloana vertebrală a autentificării în Active Directory.
- **OAuth 2.0** — un cadru de *autorizare*, nu de autentificare. Permite unui utilizator să acorde unei aplicații acces la datele sale fără să îi predea parola. **OpenID Connect** este stratul de identitate construit peste el și este componenta care face efectiv autentificarea — o distincție care se estompează constant și care contează când proiectezi un flux de login.
- **SAML** — Security Assertion Markup Language, un standard XML pentru schimbul de informații de autentificare și autorizare între un furnizor de identitate și un furnizor de servicii. Mai vechi decât OIDC și încă dominant în single sign-on-ul din companii.
- **RADIUS** — Remote Authentication Dial-In User Service (RFC 2865), care oferă autentificare, autorizare și contabilizare centralizate pentru accesul la rețea. Numele este o fosilă din era dial-up; astăzi el stă în spatele autentificării pe Wi-Fi-ul corporativ și pe VPN. Criptează doar câmpul parolei, nu întregul schimb.
- **TACACS+** — Terminal Access Controller Access-Control System Plus (RFC 8907), alternativa Cisco pentru administrarea echipamentelor. Separă autentificarea, autorizarea și contabilizarea în schimburi independente și criptează întreaga încărcătură utilă, ceea ce reprezintă principalul său avantaj față de RADIUS. TACACS simplu și XTACACS sunt ieșite din uz.

## Modele de control al accesului

Odată ce știi cine este cineva, decizi ce are voie să facă.

- **MAC (Mandatory Access Control)** — o autoritate centrală stabilește regulile, iar utilizatorii nu le pot suprascrie. Etichete și niveluri de acces; standard în sistemele guvernamentale și militare. SELinux și AppArmor sunt exemplele de zi cu zi din Linux.
- **DAC (Discretionary Access Control)** — proprietarul unei resurse decide cine primește acces. Permisiunile de fișier din Unix sunt DAC, la fel și un folder partajat pe un NAS de acasă.
- **RBAC (Role-Based Access Control)** — permisiunile se atașează rolurilor, iar utilizatorilor li se atribuie roluri. Modelul dominant în organizațiile mari, pentru că scalează cu numărul de angajați, nu cu fiecare persoană în parte.
- **ABAC (Attribute-Based Access Control)** — deciziile evaluează atribute ale utilizatorului, resursei, acțiunii și contextului, așa că poți exprima reguli de tipul „personal din financiar, pe un dispozitiv administrat, în timpul programului". Mai expresiv decât RBAC și, pe cale de consecință, mai greu de urmărit logic.

## Acces la distanță

- **VPN** — un tunel criptat peste internetul public, care oferă acces la distanță într-o rețea. VPN-urile **site-to-site** unesc permanent două rețele; VPN-urile de **client** conectează dispozitive individuale.
- **IPsec** — o suită de protocoale care securizează traficul la nivelul IP și baza majorității VPN-urilor site-to-site.
- **SSH** — acces securizat la shell la distanță peste o rețea nesigură și, totodată, transportul pentru `scp` și `sftp`.
- **RDP** — Remote Desktop Protocol, pentru acces grafic la distanță pe mașini Windows. Nu îl expune niciodată direct pe internet; pune-l în spatele unui VPN sau al unui gateway.
- **PPTP** — învechit și nesigur. Autentificarea sa MS-CHAPv2 a fost spartă în 2012, iar întregul schimb poate fi redus la o singură cheie DES. Nu îl folosi; echivalentele moderne sunt WireGuard, OpenVPN sau IKEv2/IPsec.
- **802.11** — familia de standarde IEEE pentru rețele locale *fără fir*. Folosește WPA3 sau, la limită, WPA2 cu AES-CCMP; WEP și TKIP sunt compromise.
- **RAT-urile** — remote access trojans, versiunea atacatorului pentru tot ce e mai sus: malware care oferă control de la distanță neautorizat asupra unei mașini victimă.

## Gestionarea cheilor și ciclul de viață al certificatelor

Criptografia eșuează la gestionarea cheilor mult mai des decât la matematică. Un certificat parcurge un ciclu de viață previzibil:

1. **Generarea cheilor** — entitatea solicitantă generează o pereche de chei.
2. **Depunerea identității** — își prezintă identitatea unei Autorități de Certificare (CA).
3. **Înregistrarea** — CA verifică acea identitate.
4. **Certificarea** — CA semnează un certificat care leagă cheia publică a entității de identitatea ei, folosind propria cheie privată.
5. **Distribuția** — certificatul este emis către entitate și făcut public.
6. **Utilizarea** — entitatea se autentifică și stabilește comunicații securizate.
7. **Expirarea sau revocarea** — orice certificat are o limită de timp, iar unul compromis este revocat înainte de ea.
8. **Reînnoirea** — o nouă pereche de chei și un nou certificat sunt emise la nevoie.
9. **Recuperarea** — o procedură definită pentru cazul unei chei private pierdute.
10. **Arhivarea** — certificatele și cheile sunt păstrate în siguranță pentru audit.

Un **HSM (Hardware Security Module)** este un dispozitiv fizic dedicat care generează chei, semnează și verifică, criptează și decriptează, fără ca cheia privată să părăsească vreodată hardware-ul. Furnizorii de cloud oferă echivalente administrate, iar obiectivul de design este același: să transforme extragerea cheii într-o problemă fizică, nu una software.

Două observații practice pe care ciclul de viață de mai sus nu le surprinde. Certificatele TLS publice au acum durate de viață scurte — puțin peste un an și în continuă scădere — ceea ce face reînnoirea automată obligatorie, nu opțională; ACME și Let's Encrypt există exact pentru asta. Iar revocarea este veriga slabă: atât CRL-urile, cât și OCSP au probleme binecunoscute de fiabilitate, ceea ce explică în bună măsură de ce industria a trecut la certificate cu viață scurtă.

### Gestionarea cheilor vs. gestionarea PKI

Cei doi termeni sunt folosiți interschimbabil și nu înseamnă același lucru.

| Gestionarea cheilor | Gestionarea PKI |
| --- | --- |
| Crearea, distribuirea, arhivarea și ștergerea sigură a cheilor criptografice | Producerea, distribuirea și întreținerea certificatelor digitale și a cheilor aferente |
| Cheile criptează și decriptează mesaje, autentifică utilizatori și stabilesc conexiuni sigure | Certificatele confirmă identitatea părților unei tranzacții și garantează integritatea datelor |
| Se asigură că cheile sunt folosite corect și protejate de acces neautorizat sau abuz | Stabilește încrederea între părți și revocă certificatele compromise sau invalide |

Pe scurt: gestionarea cheilor înseamnă protejarea secretelor, iar gestionarea PKI înseamnă stabilirea cui îi aparțin acele secrete.

## Atacuri

### Spoofing

Spoofing-ul înseamnă falsificarea unei identități pentru a câștiga încredere.

| Tip | Cum funcționează | Apărare |
| --- | --- | --- |
| IP spoofing | Trimiterea de pachete cu o adresă sursă falsificată | Filtrare la intrare (BCP 38) |
| Email spoofing | Falsificarea adresei `From` pentru phishing sau livrare de malware | SPF, DKIM, DMARC |
| DNS spoofing | Otrăvirea înregistrărilor DNS pentru a redirecționa traficul către serverul atacatorului | DNSSEC, DNS over HTTPS |
| Caller ID spoofing | Falsificarea unui număr de telefon pentru a imita un apelant de încredere | STIR/SHAKEN |
| MAC spoofing | Schimbarea adresei MAC a unui dispozitiv pentru a imita alt host | Autentificare pe port 802.1X |

### Machine-in-the-middle

Atacatorul se plasează între două părți și citește sau modifică trafic pe care ambele îl cred privat.

- **Deturnarea Wi-Fi** — un access point fals, adesea un dispozitiv construit special precum un Wi-Fi Pineapple, la care clienții se conectează în locul rețelei reale.
- **SSL stripping** — degradarea unei conexiuni HTTPS la HTTP, astfel încât atacatorul să o poată citi. HSTS este apărarea: îi spune browserului să nu folosească niciodată text clar pentru domeniul respectiv.
- **Deturnarea emailului** — interceptarea și modificarea corespondenței dintre două părți. Varianta comercială este business email compromise, în care detaliile facturilor sunt schimbate discret în tranzit.
- **Troienii bancari** — malware chiar pe stația utilizatorului, care urmărește sesiunile cu banca pentru a colecta credențiale și a autoriza transferuri frauduloase.

Validarea certificatelor este cea care oprește majoritatea acestor atacuri la nivel de protocol, motiv pentru care „dă pur și simplu click peste avertismentul de certificat" este un sfat mai prost decât pare.

### Denial of service

Atacurile DoS epuizează o resursă — lățime de bandă, stare a conexiunilor, CPU — până când serviciul nu mai răspunde cererilor legitime. Variantele distribuite (DDoS) generează traficul de pe multe mașini simultan.

| Atac | Mecanism |
| --- | --- |
| Ping flood | Inundă ținta cu cereri ICMP echo |
| SYN flood | Deschide conexiuni TCP neterminate până se epuizează tabela de conexiuni |
| UDP flood | Saturează rețeaua cu pachete UDP |
| HTTP flood | Trimite volume mari de cereri HTTP care par legitime |
| DNS flood | Copleșește serverele DNS ale țintei |
| Smurf attack | Trimite ICMP echo către adrese de broadcast, astfel încât răspunsurile să inunde victima |
| Amplificare NTP | Abuzează servere NTP deschise pentru a multiplica volumul de trafic către țintă |
| Slowloris | Deschide multe conexiuni și trimite lent cereri HTTP parțiale, ținând socket-urile ocupate |

Distincția care contează operațional este între atacurile **volumetrice**, care încearcă să umple conducta, și atacurile de **epuizare a resurselor** precum Slowloris, care folosesc foarte puțină lățime de bandă și consumă în schimb sloturi de conexiune. Primul tip se absoarbe cu capacitate și scrubbing în amonte; al doilea se învinge cu limite de conexiuni și timeout-uri, iar nicio cantitate de lățime de bandă nu ajută.

Atacurile de amplificare — NTP, DNS, memcached și altele — funcționează trimițând o cerere mică falsificată către un server care răspunde cu un mesaj mult mai mare, direcționat spre victimă. Ele depind atât de IP spoofing, cât și de servere publice prost configurate, motiv pentru care închiderea resolverelor deschise și filtrarea la intrare ajută pe toată lumea, nu doar pe operatorul care le implementează. Atacul Smurf este în mare parte istorie din același motiv: routerele au încetat implicit să mai transmită broadcast direcționat.
