---
date: '2025-07-03T12:18:07+03:00'
draft: false
title: 'Github Pages, Hugo si Cloudflare'
tags: ["linux", "hugo", "hosting", "cloud", "blog"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/pexels-viktortalashuk-2377295.jpg
    alt: 'Books'
---

### Cum sa rutezi site-ul tau GitHub Pages prin Cloudflare pe un subdomeniu

Ai un site gazduit pe **GitHub Pages**, de genul `https://usergithub.github.io/my-blog/`, si vrei sa-l accesezi printr-un **subdomeniu personalizat** (ex: `blog.domeniultau.ro`) folosind **Cloudflare**? 
Este o modalitate excelenta de a-ti imbunatati performanta, securitatea si de a avea un URL mai curat. Iata cum poti face asta, pas cu pas!

---

### Pasul 1: Pregateste domeniul tau in Cloudflare

Daca nu ai facut deja acest pas esential, va trebui sa iti **adaugi domeniul principal** (cum ar fi `domeniultau.ro`) in Cloudflare. Procesul implica schimbarea serverelor de nume ale domeniului la cele oferite de Cloudflare [(see docs)](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/).

Dupa ce ai modificat serverele de nume, e important sa ai putina rabdare. Propagarea DNS poate dura de la cateva minute la cateva ore. Poti verifica stadiul propagarii folosind instrumente online precum `dnschecker.org`.

---

### Pasul 2: Configureaza inregistrarea DNS in Cloudflare pentru subdomeniu

Acesta este locul unde faci legatura dintre subdomeniul tau si GitHub Pages.

1.  Acceseaza-ti contul **Cloudflare** si selecteaza domeniul principal.
2.  Mergi la sectiunea **DNS**.
3.  Vei adauga o inregistrare DNS de tip `CNAME` pentru subdomeniul tau. Aceasta va directiona traficul de la subdomeniul tau catre **domeniul de baza al GitHub Pages** (care este `usergithub.github.io`).

    * **Tip (Type):** `CNAME`
    * **Nume (Name):** Aici vei introduce **numele subdomeniului** pe care-l doresti (ex: `blog`). Subdomeniul tau complet va fi `blog.domeniultau.ro`.
    * **Tinta (Target):** `usergithub.github.io`
    * **Stare Proxy (Proxy Status):** Asigura-te ca este `Proxied` (norocul portocaliu trebuie sa fie activat). Acest lucru e crucial pentru a beneficia de toate avantajele Cloudflare: CDN, securitate, optimizari de performanta si un certificat SSL gratuit.

---

### Pasul 3: Configureaza GitHub Pages pentru domeniul personalizat

Acum trebuie sa-i spui lui GitHub ca site-ul tau va fi accesibil printr-un domeniu personalizat.

1.  Navigheaza la **repository-ul tau** (`my-blog`) pe GitHub: `https://github.com/usergithub/my-blog`.
2.  Da click pe **Settings** (Setari) in meniul repository-ului.
3.  In bara laterala din stanga, da click pe **Pages**.
4.  In sectiunea "**Custom domain**" (Domeniu Personalizat), introdu **subdomeniul complet** pe care vrei sa-l folosesti (ex: `blog.domeniultau.ro`).
5.  Da click pe **Save** (Salveaza).

    GitHub va verifica inregistrarile DNS si, daca totul este corect, va configura site-ul tau sa raspunda la adresa subdomeniului. Acest proces poate dura cateva minute.

---

### Pasul 4: Verificarea si testarea finala

* Asteapta cateva minute (sau chiar pana la o ora in cazuri rare) pentru ca toate modificarile sa se propage si sa fie recunoscute la nivel global.
* Incearca sa accesezi **subdomeniul tau** in browser (ex: `https://blog.domeniultau.ro`). Ar trebui sa vezi continutul site-ului tau.
* Verifica daca **certificatul SSL** este activ si daca site-ul se incarca prin HTTPS. Iconita cu lacat ar trebui sa fie prezenta in bara de adrese a browserului tau.

---

**Un aspect crucial de retinut:** Pentru site-urile GitHub Pages care sunt gazduite la nivel de repository, **se seteaza CNAME-ul din Cloudflare sa pointeze la `usergithub.github.io`**. Nu este necesar sa includem calea completa a repository-ului in inregistrarea CNAME. 
GitHub se va ocupa automat de maparea subdomeniului la calea corecta a repository-ului, odata ce ai specificat **"Custom domain"** in setarile GitHub Pages ale repository-ului tau.


**Optional - daca folosesti Hugo**

Actualizeaza `baseURL` in Configurarea Hugo

Acest pas este **esential** pentru ca site-ul tau Hugo sa functioneze corect sub noul subdomeniu. Daca nu-l faci, s-ar putea sa ai probleme cu link-urile, imaginile si alte resurse.

1.  Deschide fisierul de configurare al proiectului tau Hugo. De obicei, acesta se numeste `config.toml`, `config.yaml` sau `config.json` si se afla la radacina proiectului tau Hugo.
2.  Gaseste parametrul `baseURL` si actualizeaza-l cu **adresa completa a noului tau subdomeniu**, incluzand `https://`.

    **Exemplu in `config.toml`:**
    ```toml
    baseURL = https://blog.domeniultau.ro/
    ```

    **Exemplu in `config.yaml`:**
    ```yaml
    baseURL: https://blog.domeniultau.ro/
    ```
3.  **Salveaza modificarile** in repository-ul tau GitHub. GitHub Pages va reconstrui site-ul cu noua configuratie.
