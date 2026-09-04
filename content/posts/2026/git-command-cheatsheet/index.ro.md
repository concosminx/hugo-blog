---
date: '2026-09-04T09:00:00+03:00'
draft: false
title: 'Comenzi Git — Cheatsheet'
tags: ["cheatsheet", "git"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Comenzile Git care merită ținute la îndemână — configurare, ciclul zilnic de commit-uri, branch-uri și remote-uri, citirea istoricului și instrumentele de recuperare la care ajungi doar când ceva a mers prost."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

În majoritatea zilelor ai nevoie de vreo opt comenzi Git. Problema sunt celelalte patruzeci, cele care apar când un branch a ajuns unde nu trebuie, când un commit trebuie despărțit în două sau când un bug s-a strecurat undeva în ultimele două sute de commit-uri și nimeni nu mai știe când.

Ce urmează acoperă ambele jumătăți: mai întâi ciclul zilnic, apoi instrumentele de recuperare. Firul comun este că Git îți ține munca în trei locuri — working directory-ul în care editezi, index-ul (zona de staging) în care asamblezi următorul commit și repository-ul în care commit-urile devin permanente. Aproape orice comandă care pare confuză capătă sens odată ce știi pe care dintre cele trei o atinge.

## Configurare

Două setări sunt obligatorii, pentru că Git le atașează fiecărui commit și fiecărui tag pe care îl creezi:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

`--global` scrie în configurația de utilizator și se aplică peste tot. Renunță la opțiune în interiorul unui repository ca să o suprascrii doar acolo — util când commit-urile de la serviciu și cele personale au nevoie de adrese diferite.

Începând cu Git 2.28 poți alege numele pe care repository-urile noi îl dau primului branch. GitHub, GitLab și majoritatea proiectelor au trecut la `main`, iar setarea asta îți scutește o redenumire la fiecare repo nou:

```bash
git config --global init.defaultBranch main
```

Alias-urile merită cele două minute pe care le cer. Cel clasic este un graf compact al istoricului:

```bash
git config --global alias.glog "log --graph --oneline --decorate"
```

Astfel `git glog` funcționează oriunde. Încă două setări utile — editorul pe care Git îl deschide pentru mesajele de commit și pentru rebase interactiv, plus o cale directă către fișierul de configurare atunci când preferi să îl editezi manual:

```bash
git config --global core.editor vim
git config --global --edit
```

## Pornirea unui repository

```bash
git init
git init my-project
```

Fără argument, `git init` transformă directorul curent într-un repository. Cu un nume, creează întâi directorul respectiv.

Clonarea aduce un proiect împreună cu tot istoricul lui:

```bash
git clone https://github.com/user/project.git
```

Istoricul acela poate fi mare. Când vrei doar starea curentă — un agent de build, o privire rapidă peste codul altcuiva — cere o clonă superficială:

```bash
git clone --depth=1 https://github.com/user/project.git
```

Primești istoric cât un singur commit. `git log` va afișa o singură intrare, și exact asta e ideea: descărcarea este o fracțiune din dimensiune. Poți adânci ulterior cu `git fetch --unshallow` dacă te răzgândești.

## Ciclul zilnic

`git status` este comanda pe care o rulezi când nu ești sigur de nimic. Îți listează ce este modificat, ce este în staging, pe ce branch te afli și cât de mult a divergat față de remote-ul lui.

```bash
git status
git status -s
```

Forma scurtă `-s` afișează o linie per fișier și este mult mai ușor de parcurs odată ce cunoști notația.

Staging-ul este selectiv prin design. Tu alegi ce intră în următorul commit, nu comiți tot ce ai atins:

```bash
git add file.txt
git add src/
git add .
```

Înainte de staging, verifică ce urmează să incluzi. Cele două comenzi de diff răspund la întrebări diferite:

```bash
git diff
git diff --staged
```

`git diff` arată ce ai modificat, dar nu ai pus încă în staging. `git diff --staged` (scris și `--cached`) arată ce este în staging și va ajunge în următorul commit. Verificarea celei de-a doua înainte de fiecare commit prinde surprinzător de multe linii de debug rătăcite.

Apoi comiți:

```bash
git commit -m "Fix header alignment on mobile"
```

Fără `-m`, Git îți deschide editorul, ceea ce este varianta mai bună pentru orice merită un paragraf care explică *de ce*.

Ca să oprești urmărirea unui fișier și să îl ștergi într-un singur pas:

```bash
git rm file.txt
git rm --cached file.txt
```

`--cached` îl scoate din Git, dar îl lasă pe disc — soluția pentru un fișier de configurare sau un secret comis din greșeală, pe care acum vrei să îl ignori.

## Anularea muncii înainte de commit

Git-ul modern împarte treaba asta între două comenzi. `git restore` se ocupă de conținutul fișierelor, iar `git switch` de branch-uri. Ambele au fost introduse în Git 2.23 ca să înlocuiască supraîncărcatul `git checkout`, și ambele sunt stabile — `checkout` funcționează în continuare, dar comenzile mai noi sunt mult mai greu de folosit greșit.

Ca să arunci modificările necomise dintr-un fișier:

```bash
git restore file.txt
```

Operația nu se poate anula. Modificările nu au fost niciodată comise, deci Git nu are nicio copie a lor.

Ca să scoți ceva din staging fără să îți atingi modificările:

```bash
git restore --staged file.txt
```

Formele mai vechi ale acestor două comenzi sunt `git checkout -- file.txt`, respectiv `git reset file.txt`, pe care le vei mai întâlni în scripturi și în documentație mai veche.

Niciuna dintre ele nu atinge fișierele untracked. Pentru acelea ai nevoie de `git clean`, iar aici e obligatoriu să rulezi întâi în modul de probă:

```bash
git clean -nd
git clean -df
```

`-n` arată ce ar fi șters, `-d` include și directoarele, `-f` chiar execută. Să rulezi `-df` fără să verifici întâi cu `-nd` este o metodă sigură de a pierde o oră de muncă ce nu a fost niciodată în Git.

## Stash

Când trebuie să schimbi contextul, dar munca nu e gata de commit:

```bash
git stash
git stash pop
```

`git stash` îți pune deoparte modificările necomise și îți lasă un working directory curat. `git stash pop` le aduce înapoi și șterge intrarea din stash. Stash-urile formează o stivă, așa că le poți lista și le poți arunca pe cele de care nu mai ai nevoie:

```bash
git stash list
git stash drop stash@{1}
```

## Branch-uri

```bash
git branch
git branch -a
```

`git branch` simplu listează branch-urile locale; `-a` adaugă și pe cele de urmărire de la distanță. Adaugă `-v` ca să vezi ultimul commit al fiecărui branch și cum stă față de upstream-ul lui.

Creare și comutare:

```bash
git switch -c feature/login
git switch main
```

`-c` creează branch-ul și te mută pe el. Fără el comuți pe un branch care există deja. Echivalentele cu `checkout` sunt `git checkout -b feature/login` și `git checkout main`.

Merge-ul aduce alt branch în cel pe care te afli:

```bash
git switch main
git merge feature/login
```

Ștergerea este intenționat incomodă atunci când ar putea pierde muncă:

```bash
git branch -d feature/login
git branch -D feature/login
```

`-d` refuză dacă branch-ul are commit-uri care nu au fost integrate nicăieri. `-D` forțează. Folosește `-d` implicit și lasă-l să te oprească.

După un proiect lung, branch-urile locale se adună. Ca să faci curat peste tot în afară de `main`:

```bash
git branch | grep -v "main" | xargs git branch -D
```

Asta este o ștergere forțată pe tot ce se potrivește, așa că rulează întâi doar partea `git branch | grep -v "main"` și citește lista.

Redenumirea unui branch — inclusiv trecerea de la `master` la `main` într-un proiect mai vechi — se face cu `-m`:

```bash
git branch -m master main
```

Dacă branch-ul există și pe un remote, redenumește-l și în interfața platformei de hosting, apoi reindică branch-ul local cu `git push -u origin main`.

## Remote-uri

Un remote este un URL cu nume. `origin` este numele convențional pentru cel din care ai clonat, dar nu are nimic special:

```bash
git remote -v
git remote add upstream https://github.com/original/project.git
git remote get-url origin
```

Git este distribuit, așa că poți avea oricâte remote-uri vrei și poți face push către oricare dintre ele:

```bash
git remote add working https://gitlab.com/user/mirror.git
git push working main
```

Fetch-ul descarcă commit-uri noi fără să îți schimbe working directory-ul:

```bash
git fetch origin
git fetch --prune origin
```

`--prune` șterge referințele locale către branch-uri remote care nu mai există în amonte — merită adăugat ori de câte ori lista ta de branch-uri pare învechită.

`git pull` este un fetch urmat imediat de un merge în branch-ul curent:

```bash
git pull origin main
git pull --rebase origin main
```

`--rebase` îți rejoacă commit-urile locale peste ce ai adus, în loc să creeze un commit de merge, ceea ce păstrează istoricul liniar.

Push-ul trimite commit-urile în cealaltă direcție:

```bash
git push origin main
git push -u origin feature/login
git push --tags origin
```

`-u` setează upstream-ul, astfel încât următoarele `git push` și `git pull` pe acel branch nu mai au nevoie de argumente.

Push-ul forțat rescrie branch-ul de pe remote și poate distruge commit-urile altcuiva. Când chiar ai nevoie de el — de obicei după un rebase interactiv pe un branch folosit doar de tine — preferă forma mai sigură:

```bash
git push --force-with-lease origin feature/login
```

`--force-with-lease` refuză dacă remote-ul are commit-uri pe care nu le-ai văzut, adică exact situația în care `--force` simplu ar șterge în tăcere munca unui coleg.

## Menținerea unui fork la zi

Un fork nu se actualizează singur. Când proiectul original merge mai departe, fork-ul tău rămâne unde era. Soluția este un al doilea remote, numit prin convenție `upstream`, care indică spre proiectul din care ai făcut fork:

```bash
git remote add upstream https://github.com/original/project.git
git remote -v
```

Ar trebui să vezi acum `origin` (fork-ul tău) și `upstream` (originalul), fiecare listat pentru fetch și push. Aducerea fork-ului la zi înseamnă apoi un fetch și un merge:

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```

Fă asta înainte să începi orice ramură nouă de lucru. Un rebase al unui branch vechi de o săptămână peste o țintă care s-a mutat este considerabil mai neplăcut decât pornirea de la unul actualizat.

## Citirea istoricului

`git log` în forma implicită este stufos. Acestea patru merită memorate:

```bash
git log --oneline
git log -5
git log --oneline --graph --decorate
git log -p
```

Un commit per linie; ultimele cinci commit-uri; un graf text cu etichetele de branch-uri și tag-uri atașate; și diff-ul complet al fiecărui commit.

Filtrarea este momentul în care log devine un instrument de căutare, nu un zid de text:

| Comandă | Ce găsește |
| --- | --- |
| `git log --oneline --after="24 months ago"` | Commit-uri de la o dată încoace, în engleză simplă sau în format ISO |
| `git log --oneline --before="2026-01-01"` | Celălalt capăt al intervalului |
| `git log --grep="[Aa]nimation"` | Commit-uri al căror mesaj se potrivește cu un tipar |
| `git log --author="Maria"` | Commit-urile unei singure persoane |
| `git log -- src/index.html` | Commit-urile care au atins un anumit fișier |
| `git log --stat` | Ce fișiere s-au schimbat, cu numărul de linii |
| `git log main..feature` | Commit-uri de pe `feature` care nu sunt încă în `main` |

Acestea se combină, iar acolo stă valoarea reală. Un istoric concis al muncii pe animații din ultimii doi ani înseamnă o singură comandă:

```bash
git log --oneline --after="24 months ago" --grep="[Aa]nimation"
```

Ca să vezi un singur commit în întregime, inclusiv diff-ul lui:

```bash
git show a1b2c3d
```

Iar ca să afli cine a modificat ultima dată fiecare linie dintr-un fișier și în ce commit:

```bash
git blame build/index.html
git blame -s build/index.html
```

`-s` ascunde numele autorului și data, lăsând doar hash-urile commit-urilor — o vedere mai strânsă când vrei doar să sari la commit-uri.

## Tag-uri

Tag-urile marchează un commit ca fiind important, de obicei o versiune:

```bash
git tag
git tag v2.0.1
git tag -a v2.0.1 -m "Release 2.0.1"
git tag -d v2.0.1
```

`git tag <nume>` simplu creează un tag lightweight, un simplu pointer către un commit. `-a` creează un tag adnotat, un obiect real cu autor, dată și mesaj. Folosește tag-uri adnotate pentru orice publici; cele lightweight sunt potrivite ca semne de carte private.

Adaugă un hash de commit ca să marchezi altceva decât `HEAD`:

```bash
git tag v1.0.0 a1b2c3d
```

Tag-urile nu sunt trimise printr-un `git push` obișnuit. Trimite-le explicit cu `git push --tags origin`, cum s-a arătat mai sus.

## Rescrierea istoricului

Tot ce urmează în această secțiune modifică commit-uri care există deja. Asta este sigur pe commit-uri pe care nu le-ai distribuit și deranjant pe commit-uri pe care alții le-au adus deja la ei.

Ca să repari commit-ul tocmai făcut:

```bash
git commit --amend -m "A better message for the previous commit"
git commit --amend --no-edit
```

Prima variantă înlocuiește ultimul commit cu modificările din staging plus un mesaj nou. A doua adaugă modificările din staging în ultimul commit, lăsându-i mesajul neschimbat — mișcarea standard când ai uitat un fișier.

Rebase-ul interactiv este instrumentul pentru a face ordine într-o serie de commit-uri înainte de a deschide un pull request:

```bash
git rebase -i HEAD~4
```

Comanda deschide ultimele patru commit-uri în editor, câte unul per linie, fiecare precedat de o instrucțiune. Schimbă `pick` în `squash` ca să topești un commit în cel de deasupra, în `reword` ca să îi editezi mesajul, în `drop` ca să îl ștergi, sau rearanjează liniile ca să reordonezi commit-urile. Așa ajunge un branch care conține „fix typo", „fix typo again" și „actually fix it" un singur commit coerent.

Un rebase obișnuit îți mută branch-ul peste o bază nouă:

```bash
git rebase main
```

## Mutarea unor commit-uri anume între branch-uri

Când vrei două-trei commit-uri de pe alt branch, nu tot branch-ul, folosește cherry-pick:

```bash
git switch main
git cherry-pick a1b2c3d e4f5g6h
```

Fiecare este aplicat pe branch-ul curent ca un commit nou. Adăugarea lui `-n` aplică modificările în working directory și în index fără să comită, ca să le poți revizui sau combina înainte de a face un singur commit:

```bash
git cherry-pick a1b2c3d e4f5g6h -n
```

Ia hash-urile întâi cu `git log --oneline` pe branch-ul sursă.

## Anularea muncii deja comise

Două comenzi fac asta, iar diferența contează.

`git revert` creează un commit *nou* care anulează unul vechi:

```bash
git revert a1b2c3d
```

Nu se șterge nimic, deci este alegerea sigură pentru orice a fost deja trimis cu push.

`git reset` mută pointerul branch-ului înapoi, iar cele trei moduri ale lui diferă prin cât iau cu ele:

```bash
git reset --soft HEAD~1
git reset HEAD~1
git reset --hard HEAD~1
```

`--soft` mută doar branch-ul, lăsându-ți modificările în staging. Modul implicit `--mixed` resetează și index-ul, lăsând modificările în working directory ca editări nepuse în staging. `--hard` resetează și working directory-ul și aruncă tot ce era după acel commit.

### Recuperarea după un reset greșit

`git reset --hard` pare definitiv, dar de cele mai multe ori nu este. Git ține un jurnal al tuturor pozițiilor prin care a trecut `HEAD`, inclusiv cele pe care le-ai abandonat prin reset:

```bash
git reflog
```

Fiecare linie are un hash și o descriere a ce a mutat `HEAD` acolo. Găsește starea pe care o vrei înapoi și fă reset la ea:

```bash
git reset --hard a1b2c3d
```

Reflog-ul este local, nu se distribuie și expiră implicit după nouăzeci de zile. A salvat mai multă muncă decât orice altă comandă Git și merită verificat înainte să tragi concluzia că ceva s-a pierdut.

## Găsirea commit-ului care a stricat ceva

Când un bug există acum și nu exista la un moment dat în trecut, `git bisect` găsește commit-ul vinovat prin căutare binară. Douăzeci de commit-uri cer vreo patru testări; o mie cer vreo zece.

```bash
git bisect start
git bisect bad
git bisect good a1b2c3d
```

Marchezi starea curentă ca fiind proastă și un commit despre care știi că era bun, luat din `git log --oneline`. Git face checkout la un commit aflat la jumătatea drumului. Testează-l, apoi spune-i lui Git ce ai găsit:

```bash
git bisect good
git bisect bad
```

Repetă până când Git îți numește primul commit problematic. Apoi încheie sesiunea și întoarce-te de unde ai plecat:

```bash
git bisect reset
```

Dacă uiți `git bisect reset`, rămâi pe un checkout detașat în mijlocul istoricului, o stare confuză în care nu vrei să te trezești mai târziu.

Dacă bug-ul poate fi detectat de un script sau de un test, dă-i lui Git toată treaba:

```bash
git bisect run npm test
```

Git rulează comanda la fiecare pas și îi folosește codul de ieșire — zero pentru bun, diferit de zero pentru prost — ca să conducă singur căutarea.

## Lucrul pe două branch-uri în același timp

Stash-uitul și comutatul devin obositoare când ai nevoie de două branch-uri unul lângă altul — ca să compari comportamente, sau ca să repari un bug în timp ce rulează un build lung. Un worktree îți dă un al doilea director cu checkout, susținut de același repository:

```bash
git worktree add ../project-hotfix
git worktree list
cd ../project-hotfix
```

`git worktree add` creează directorul și face checkout la un branch acolo. Ambele directoare împart același `.git`, așa că un commit făcut în oricare dintre ele este imediat vizibil în celălalt și nu ai o a doua clonă de ținut sincronizată. Fă curat când ai terminat:

```bash
git worktree remove ../project-hotfix
```

## Exportarea unui snapshot

`git archive` împachetează arborele unui commit ca arhivă, fără director `.git` și fără istoric:

```bash
git archive -o ../release.zip main
git archive -o ../build.zip feature/login -- build
```

A doua formă limitează arhiva la o singură cale. Aceasta este metoda corectă de a-i da cuiva o copie a codului fără repository și este mai bună decât copierea manuală a folderului — conținutul vine din Git, deci fișierele ignorate și cele untracked nu ajung niciodată acolo.

## Ignorarea fișierelor

Un fișier `.gitignore` în rădăcina repository-ului ține fișierele generate și pe cele locale în afara lui `git status` și în afara commit-urilor:

```text
/logs/*
!logs/.gitkeep
/tmp
*.swp
.env
```

Tiparele sunt relative la directorul propriu al fișierului și se aplică și subdirectoarelor. Un `!` la început reinclude ceva exclus de un tipar anterior — mai sus, directorul `logs/` rămâne în repository prin `.gitkeep`, în timp ce conținutul lui este ignorat.

`.gitignore` afectează doar fișierele untracked. Un fișier pe care Git îl urmărește deja rămâne urmărit indiferent ce adaugi în listă; folosește `git rm --cached` ca să oprești asta, cum am descris mai devreme.

## Unde să cauți mai departe

`git help <comandă>` deschide manualul complet pentru orice comandă de aici, iar referința oficială de la [git-scm.com/docs](https://git-scm.com/docs) este chiar bună. Pentru modelul din spate, nu doar pentru comenzi, *Pro Git* de Scott Chacon și Ben Straub este gratuit online la [git-scm.com/book](https://git-scm.com/book) și rămâne cea mai bună explicație a motivului pentru care Git funcționează așa cum funcționează.
