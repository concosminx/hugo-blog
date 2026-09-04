---
date: '2026-09-04T10:00:00+03:00'
draft: false
title: 'Big O: Complexitatea de Timp și de Spațiu Explicată'
taguri: ["cheatsheet", "algorithms"]
categorii: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Clasele de complexitate, regulile pentru simplificarea lor și ce consumă de fapt timp și memorie într-o funcție."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

Big O descrie cum crește costul unui algoritm pe măsură ce crește dimensiunea datelor de intrare. Nu cât de rapid este pe laptopul tău — ci cum *scalează*. O funcție O(n²) poate să bată una O(n log n) pe zece elemente și să piardă catastrofal pe zece milioane, iar rostul notației este exact să facă acest punct de răsturnare previzibil, fără să rulezi niciun benchmark.

## Clasele de complexitate

De la cea mai bună la cea mai proastă, acestea sunt cele care merită memorate:

| Notație | Nume | Sursă tipică |
| --- | --- | --- |
| O(1) | Constantă | Fără bucle — indexare în array, căutare în hash, aritmetică |
| O(log n) | Logaritmică | Înjumătățirea spațiului de căutare la fiecare pas — căutare binară, arbore echilibrat |
| O(n) | Liniară | O singură parcurgere a n elemente |
| O(n log n) | Liniar-logaritmică | Sortări prin comparație — merge sort, heapsort, Timsort |
| O(n²) | Pătratică | Două bucle imbricate peste aceeași colecție |
| O(2ⁿ) | Exponențială | Recursivitate naivă care se ramifică de două ori la fiecare apel |
| O(n!) | Factorială | Generarea tuturor permutărilor |

**O(1)** înseamnă că munca nu depinde deloc de dimensiunea intrării. Accesarea lui `users[500]` costă la fel indiferent dacă array-ul are o mie de elemente sau un miliard.

**O(log n)** apare ori de câte ori fiecare pas elimină o fracțiune fixă din munca rămasă. Căutarea binară este cazul clasic și cere ca datele să fie deja sortate — sortarea lor costă O(n log n), așa că o singură căutare binară pe un array nesortat nu reprezintă un câștig.

**O(n)** este o singură buclă prin n elemente. Este și pragul minim pentru orice problemă în care chiar trebuie să te uiți la fiecare element: nu poți găsi maximul dintr-un array nesortat în mai puțin de O(n), pentru că sărind peste orice element riști să sari exact peste răspuns.

**O(n log n)** este plafonul practic pentru sortarea de uz general. Nicio sortare care funcționează comparând perechi de elemente nu poate face mai bine — este o limită inferioară demonstrată matematic, nu doar o limitare de inginerie. Sortările care o depășesc, precum counting sort sau radix sort, reușesc pentru că nu compară deloc elementele, ceea ce înseamnă că funcționează doar pe chei cu structură restrânsă.

**O(n²)** este locul unde ajung buclele imbricate naive: compararea fiecărui element cu toate celelalte. Acceptabil pentru o sută de elemente, dureros la zece mii, fără speranță la un milion.

**O(2ⁿ)** și **O(n!)** sunt clasele în care dimensiunea intrării încetează să conteze pentru că nimic nu mai este tratabil. Fibonacci recursiv naiv este O(2ⁿ) pentru că fiecare apel generează alte două; rezolvarea prin forță brută a problemei comis-voiajorului încercând toate rutele este O(n!). Dacă algoritmul tău se află într-una dintre aceste clase, soluția este un algoritm diferit — memoizare, programare dinamică sau acceptarea unei aproximări.

## Ce consumă de fapt timp

Când numeri operațiile dintr-o funcție, acestea sunt lucrurile care contează:

- **Aritmetica** — `+`, `-`, `*`, `/`
- **Comparațiile** — `<`, `>`, `==`
- **Buclele** — `for`, `while` și orice formă de iterație construită peste ele
- **Apelurile de funcții** — inclusiv cele pe care nu le-ai scris tu

Ultima este capcana. Un apel către o funcție externă nu este gratuit și nu este O(1) doar pentru că ocupă o singură linie în codul tău. O sortare într-o buclă sau un helper care parcurge discret o listă — acolo se transformă o funcție aparent O(n) în O(n²).

```python
def has_duplicates(items):
    for item in items:
        if items.count(item) > 1:   # count() este ea însăși O(n)
            return True
    return False
```

Codul arată ca o singură buclă, deci pare liniar. Este O(n²), pentru că `count()` parcurge întreaga listă la fiecare iterație. Soluția este un set, care schimbă O(n) spațiu pentru O(n) timp.

## Cele patru reguli

Big O este o simplificare, iar pentru simplificare există patru reguli.

### Regula 1: Presupune întotdeauna cazul cel mai defavorabil

Big O descrie limita superioară. Dacă cauți o valoare într-o listă și ea se nimerește să fie la indexul 0, acea rulare a fost O(1) — dar căutarea liniară rămâne O(n), pentru că trebuie să presupui că valoarea este la final sau lipsește complet.

```python
def find(items, target):
    for item in items:       # cazul cel mai rău: toate elementele
        if item == target:
            return True
    return False
```

De aceea „de obicei e rapid" nu este o afirmație despre complexitate. Complexitatea se referă la cazul care doare.

### Regula 2: Elimină constantele

O(2n) este O(n). O(n/2) este O(n). O funcție cu două bucle secvențiale peste același array rămâne liniară — dublarea muncii nu schimbă modul în care scalează, ci doar panta.

Corolarul îi surprinde pe mulți: **parcurgerea a jumătate dintr-o colecție este tot O(n)**. Înjumătățirea factorului constant este o accelerare reală pe hardware real, dar este invizibilă pentru Big O.

### Regula 3: Intrări diferite primesc variabile diferite

Dacă o funcție primește două colecții, ele au nevoie de două variabile. Numindu-le pe amândouă „n" ascunzi comportamentul real:

```python
def compare(a, b):
    for x in a:          # O(a)
        pass
    for y in b:          # O(b)
        pass
    # total: O(a + b)

def cross(a, b):
    for x in a:
        for y in b:      # rulează len(a) * len(b) ori
            pass
    # total: O(a * b)
```

Prescurtarea merită memorată:

- **`+` pentru pași în secvență** — o buclă după alta
- **`*` pentru pași imbricați** — o buclă în interiorul alteia

Două colecții separate parcurse una după alta înseamnă O(a + b). Aceleași două colecții imbricate înseamnă O(a × b). Doar când ambele sunt aceeași colecție al doilea caz se reduce la familiarul O(n²).

### Regula 4: Elimină termenii nedominanți

O(n² + n) este O(n²). Pe măsură ce n crește, termenul pătratic îneacă tot restul, iar termenii mai mici încetează să conteze. Păstrează doar termenul care crește cel mai repede.

Singura rezervă vine din Regula 3: O(a² + b) **nu** se simplifică la O(a²), pentru că a și b sunt independente. Nu poți elimina un termen decât dacă știi că este dominat de altul, iar intrările separate nu îți oferă nicio astfel de garanție.

## Complexitatea de spațiu

Aceeași notație se aplică și memoriei, iar regulile rămân aceleași. Ce consumă spațiu:

- **Variabilele** pe care le declari
- **Structurile de date** pe care le aloci
- **Apelurile de funcții** — fiecare ocupă un stack frame
- **Alocările** în general, inclusiv copiile pe care nu ai intenționat să le faci

Intrarea legată de apelurile de funcții contează mai mult decât pare. O recursivitate care coboară n niveluri consumă O(n) spațiu chiar dacă nu alocă nimic, pentru că n stack frame-uri sunt active simultan. Exact așa ajunge o parcurgere recursivă a unei liste înlănțuite mari să producă un stack overflow, în timp ce bucla echivalentă rulează liniștită în O(1) spațiu.

Timpul și spațiul se negociază frecvent unul pe seama celuilalt. Verificarea duplicatelor de mai sus trece de la O(n²) timp și O(1) spațiu la O(n) timp și O(n) spațiu dacă păstrezi un set cu ce ai văzut deja. Care compromis este cel corect depinde de resursa care îți lipsește.

## Dincolo de cazul cel mai defavorabil

Două nuanțe apar suficient de des încât să merite cunoscute.

**Big O este doar limita superioară.** Θ (theta) descrie o limită strânsă — rata de creștere atât deasupra, cât și dedesubt — iar Ω (omega) descrie limita inferioară. Majoritatea conversațiilor de lucru spun „Big O" când de fapt se referă la theta; asta este inofensiv atâta timp cât toată lumea știe că distincția există.

**Complexitatea amortizată** acoperă operațiile care sunt de obicei ieftine, dar ocazional scumpe. Adăugarea la finalul unui array dinamic este O(1) în majoritatea cazurilor și O(n) atunci când array-ul trebuie să crească și să se copieze. Mediată pe o serie lungă de adăugări, operația costă O(1) amortizat — o afirmație cu adevărat utilă și diferită de O(n) în cazul cel mai defavorabil.

## Operații uzuale dintr-o privire

Complexitățile pentru structurile la care ajungi zilnic. Cifrele pentru hash table sunt cazul mediu; cazul cel mai defavorabil degradează la O(n) când toate cheile intră în coliziune.

| Operație | Array | Array dinamic | Listă înlănțuită | Hash table | Arbore binar echilibrat |
| --- | --- | --- | --- | --- | --- |
| Acces după index | O(1) | O(1) | O(n) | — | — |
| Căutare după valoare | O(n) | O(n) | O(n) | O(1) | O(log n) |
| Inserare la final | — | O(1) amortizat | O(1) | O(1) | O(log n) |
| Inserare la început | — | O(n) | O(1) | — | O(log n) |
| Ștergere după valoare | — | O(n) | O(n) | O(1) | O(log n) |

Tiparul din spatele tabelului este partea utilă: memoria contiguă îți cumpără indexare ieftină și inserare scumpă, pointerii îți cumpără exact invers, iar hashing-ul îți cumpără totul ieftin, cu prețul ordinii și al garanțiilor în cazul cel mai defavorabil. Alegerea unei structuri de date înseamnă, în mare parte, alegerea coloanei pe care o vrei.
