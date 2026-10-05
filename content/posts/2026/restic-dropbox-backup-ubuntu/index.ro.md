---
date: '2026-10-05T18:00:00+03:00'
draft: false
title: 'Backup săptămânal criptat în Dropbox cu restic și rclone pe Ubuntu 24.04'
tags: ["linux", "ubuntu", "backup"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Configurează backup-uri săptămânale criptate și deduplicate de pe Ubuntu 24.04 în Dropbox, folosind restic, rclone și un systemd user timer — inclusiv cum restaurezi datele când ai nevoie de ele."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

Un backup din care nu ai restaurat niciodată este doar un zvon. Aceasta este configurația pe care o rulez pe un server de casă: **restic** se ocupă de criptare și deduplicare, **rclone** îi oferă o cale către Dropbox, iar un **systemd user timer** rulează totul în fiecare duminică seara. Tot ce pleacă de pe mașină este criptat local, așa că Dropbox stochează doar text cifrat.

A doua jumătate a articolului este despre cum scoți datele înapoi, partea pe care majoritatea ghidurilor o tratează superficial.

## Ce face această configurație

Odată pornită, în fiecare duminică la 20:00 serverul se trezește, urcă ce s-a schimbat de săptămâna trecută, taie snapshot-urile vechi conform unei politici de retenție și se culcă din nou. Dacă mașina era oprită la ora respectivă, backup-ul rulează la următoarea pornire în loc să fie sărit.

Câteva lucruri de știut înainte să începi:

- Backup-urile sunt **criptate cu o parolă creată de tine**. Dropbox nu le poate citi — și nici tu, dacă pierzi parola.
- Datele sunt **deduplicate**, așa că a doua rulare săptămânală urcă doar blocurile modificate, nu tot de la început.
- Repository-ul stă în Dropbox, dar este accesat exclusiv prin rclone. **Nu îl pune într-un folder sincronizat de clientul desktop Dropbox** — două procese care scriu în același repository este exact modul în care repository-urile se corup.

## Instalează restic și rclone

Ambele unelte există în arhiva Ubuntu, dar pe 24.04 sunt suficient de învechite ca să conteze:

```bash
sudo apt install restic
restic version
```

Ubuntu 24.04 vine cu **restic 0.16.4**, în timp ce upstream a trecut demult de 0.19. Diferența nu este cosmetică — comanda `restore` mult îmbunătățită, inclusiv modurile `--dry-run` și suprascrierea pe loc, a apărut în **restic 0.17.0**. Dacă vrei opțiunile de restaurare descrise mai jos, instalează binarul din release-ul curent:

```bash
sudo apt install bzip2
RESTIC_VER=$(curl -s https://api.github.com/repos/restic/restic/releases/latest | grep -oP '"tag_name": "v\K[^"]+')
wget "https://github.com/restic/restic/releases/latest/download/restic_${RESTIC_VER}_linux_amd64.bz2" -O restic.bz2
bunzip2 restic.bz2 && chmod +x restic && sudo mv restic /usr/local/bin/
restic version
```

Astfel binarul ajunge în `/usr/local/bin`, care are prioritate față de `/usr/bin` în `PATH`-ul implicit, deci câștigă în fața copiei instalate din apt. Îl ții la zi cu `sudo restic self-update`.

rclone are aceeași problemă, așa că ia-l din scriptul oficial de instalare, nu din apt:

```bash
sudo -v ; curl https://rclone.org/install.sh | sudo bash
rclone version
```

Scriptul verifică ce este deja instalat și sare peste descărcare dacă ești la zi, ceea ce îl face sigur de rulat din nou mai târziu.

## Conectează rclone la Dropbox

Remote-ul se creează interactiv:

```bash
rclone config
```

Parcurge pașii astfel:

1. `n` pentru un remote nou și numește-l **`dropbox`** — numele contează, pentru că devine parte din calea repository-ului.
2. Tipul de storage: `dropbox`.
3. `client_id` și `client_secret`: lasă-le ambele goale pentru a folosi credențialele interne ale rclone. Poți înregistra propria aplicație în [Dropbox App Console](https://www.dropbox.com/developers/apps) și să îi dai cheia și secretul; rclone documentează exact permisiunile necesare în [documentația backend-ului Dropbox](https://rclone.org/dropbox/).
4. Acceptă restul valorilor implicite, răspunde da la auto config și finalizează autentificarea în browser-ul care se deschide.

Pe un **server headless** nu există browser care să se deschidă. Rulează `rclone authorize "dropbox"` pe o mașină care are unul, apoi lipește token-ul rezultat în prompt-ul de configurare al serverului — procedura completă este în [documentația de remote setup](https://rclone.org/remote_setup/).

Verifică faptul că remote-ul funcționează înainte să continui:

```bash
rclone lsd dropbox:
```

Comanda listează directoarele de la rădăcina Dropbox-ului. Dacă dă eroare, rezolvă aici — nimic din ce urmează nu va funcționa până când această comandă nu merge.

## Creează parola și inițializează repository-ul

> ⚠️ **Dacă pierzi această parolă, toate backup-urile devin definitiv ilizibile.** Nu există niciun mecanism de recuperare, prin design. Pune o copie într-un password manager înainte să rulezi orice altceva.

```bash
mkdir -p ~/.config/restic
echo 'a-long-random-password' > ~/.config/restic/password
chmod 600 ~/.config/restic/password
```

Folosește ceva lung și aleatoriu — `openssl rand -base64 32` este o sursă rezonabilă. `chmod 600` face fișierul citibil doar de utilizatorul tău.

restic preia locația repository-ului și parola din două variabile de mediu, ceea ce te scutește să le repeți la fiecare comandă:

```bash
export RESTIC_REPOSITORY="rclone:dropbox:restic-backups-homeserver"
export RESTIC_PASSWORD_FILE="$HOME/.config/restic/password"
restic init
```

Șirul repository-ului se citește ca `rclone:<remote>:<cale>`, deci comanda creează un folder numit `restic-backups-homeserver` la rădăcina Dropbox-ului. Numește-l după mașină dacă ai de gând să faci backup pentru mai multe — fiecare host își vrea propriul repository, dacă nu ai un motiv anume să îl partajezi.

`restic init` scrie structura repository-ului și un fișier `config`. Este o operație unică; rulată peste un repository existent, este refuzată, nu distructivă.

Ca să funcționeze comenzile `restic` simple în shell-ul tău interactiv, adaugă cele două linii `export` în `~/.bashrc` și rulează `source ~/.bashrc`.

## Scrie scriptul de backup

Job-ul programat este un simplu script de shell, ceea ce îl face testabil manual:

```bash
mkdir -p ~/.local/bin
cat > ~/.local/bin/restic-backup.sh << 'EOF'
#!/bin/bash
set -euo pipefail

export RESTIC_REPOSITORY="rclone:dropbox:restic-backups-homeserver"
export RESTIC_PASSWORD_FILE="$HOME/.config/restic/password"

echo "=== Backup started: $(date) ==="

restic backup \
  "$HOME/Documents" \
  "$HOME/Pictures" \
  "$HOME/Projects" \
  --exclude-caches \
  --exclude "$HOME/.cache" \
  --exclude "node_modules" \
  --exclude "*.tmp"

restic forget \
  --keep-weekly 8 \
  --keep-monthly 12 \
  --keep-yearly 2 \
  --prune

echo "=== Backup finished: $(date) ==="
EOF
chmod +x ~/.local/bin/restic-backup.sh
```

Punerea delimitatorului heredoc între ghilimele simple, `'EOF'`, este intenționată: împiedică shell-ul să expandeze `$HOME` și `$(date)` în momentul scrierii fișierului, așa că acestea sunt evaluate când rulează scriptul, nu înghețate la valorile de azi.

`set -euo pipefail` face scriptul să se oprească la prima eroare. Ordinea contează — dacă `restic backup` eșuează, nu vrei ca `restic forget --prune` să ruleze după el și să șteargă snapshot-uri vechi pentru a face loc unui backup care nu a ajuns niciodată.

Două comenzi fac treaba. `restic backup` parcurge căile listate și urcă blocurile noi sau modificate; `--exclude-caches` respectă convenția `CACHEDIR.TAG`, iar restul opțiunilor `--exclude` sar peste gunoiul obișnuit. Apoi `restic forget` aplică politica de retenție — opt snapshot-uri săptămânale, douăsprezece lunare și două anuale — iar `--prune` este cel care eliberează efectiv spațiul din Dropbox. Fără `--prune`, `forget` șterge doar etichetele snapshot-urilor, iar datele rămân pe loc.

**Modifică lista de căi ca să se potrivească mașinii tale.** O cale care nu există face toată rularea să eșueze:

```bash
nano ~/.local/bin/restic-backup.sh
```

Apoi rulează-l o dată manual, înainte să implici systemd:

```bash
~/.local/bin/restic-backup.sh
```

Prima rulare urcă tot și va dura. Rulările următoare sunt incrementale.

## Programează-l cu un systemd user timer

Două fișiere unit: un service care spune ce să ruleze și un timer care spune când.

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/restic-backup.service << 'EOF'
[Unit]
Description=Restic backup to Dropbox
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=%h/.local/bin/restic-backup.sh
Nice=10
IOSchedulingClass=idle
EOF

cat > ~/.config/systemd/user/restic-backup.timer << 'EOF'
[Unit]
Description=Weekly restic backup (Sunday 20:00)

[Timer]
OnCalendar=Sun *-*-* 20:00 Europe/Bucharest
Persistent=true
RandomizedDelaySec=10min

[Install]
WantedBy=timers.target
EOF
```

În service, `Type=oneshot` îi spune systemd că procesul rulează până la capăt și iese, în loc să rămână rezident. `Wants=` și `After=network-online.target` îl țin pe loc până când rețeaua este efectiv funcțională. `Nice=10` și `IOSchedulingClass=idle` îl trimit la coada de prioritate pentru CPU și disc, așa că un backup nu face niciodată mașina să pară lentă. `%h` se expandează la directorul home.

Timer-ul nu are nevoie de o linie `Unit=`, pentru că un timer activează implicit service-ul cu același nume. Partea interesantă este fusul orar de la final:

```ini
OnCalendar=Sun *-*-* 20:00 Europe/Bucharest
```

Adăugarea unui nume de fus orar [IANA](https://www.iana.org/time-zones) fixează programul la ora locală din România, indiferent de cum este configurat ceasul serverului, și urmărește corect trecerea la ora de vară. Ubuntu 24.04 vine cu systemd 255, confortabil peste versiunea v250 care a introdus sufixele de fus orar — pe ceva mai vechi, renunță la fusul orar și setează în schimb ceasul sistemului.

`Persistent=true` ține minte ultima rulare, așa că o ocazie ratată se declanșează la următoarea pornire, în loc să fie pierdută. `RandomizedDelaySec=10min` împrăștie ora de start pe un interval de zece minute, ceea ce este o formă de politețe față de API-ul Dropbox dacă rulezi asta pe mai multe mașini.

Înainte să activezi orice, pune systemd să verifice expresia și să îți spună când se declanșează următoarea dată:

```bash
systemd-analyze calendar "Sun *-*-* 20:00 Europe/Bucharest"
```

Apoi încarcă și activează:

```bash
systemctl --user daemon-reload
systemctl --user enable --now restic-backup.timer
sudo loginctl enable-linger $USER
systemctl --user list-timers restic-backup.timer
```

`daemon-reload` recitește fișierele unit — rulează-l după fiecare modificare, altfel systemd continuă să folosească versiunea veche. `enable --now` activează timer-ul permanent, astfel încât supraviețuiește reboot-urilor, și îl pornește imediat.

`loginctl enable-linger` este pasul pe care lumea îl ratează și este **obligatoriu pe un server**. Fără el, instanța systemd a utilizatorului tău este oprită când se închide ultima sesiune, deci timer-ul moare în momentul în care închizi SSH-ul. Verifică că a prins:

```bash
loginctl show-user $USER | grep Linger
```

Ar trebui să afișeze `Linger=yes`.

## Comenzi de zi cu zi

| Comandă | Ce face |
| --- | --- |
| `systemctl --user start restic-backup` | Rulează un backup chiar acum |
| `systemctl --user list-timers` | Arată următoarea și ultima rulare |
| `journalctl --user -u restic-backup -e` | Citește log-urile (`-f` pentru urmărire live, `q` pentru ieșire) |
| `restic snapshots` | Listează toate snapshot-urile din repository |
| `restic stats` | Arată dimensiunea repository-ului și economia din deduplicare |
| `restic check` | Verifică integritatea repository-ului |

Comenzile `restic` au nevoie de `RESTIC_REPOSITORY` și `RESTIC_PASSWORD_FILE` în mediu. Dacă nu le-ai adăugat în `~/.bashrc`, transmite-le la fiecare comandă:

```bash
restic -r rclone:dropbox:restic-backups-homeserver \
  --password-file ~/.config/restic/password snapshots
```

## Restaurarea

Aceasta este jumătatea care merită repetată. Fă-o o dată, în mod deliberat, cât timp nu arde nimic.

### Găsește ce vrei

Orice restaurare pornește de la un ID de snapshot. Listează-le:

```bash
restic snapshots
```

Primești un ID, o dată, host-ul și căile acoperite de fiecare snapshot. Cuvântul `latest` funcționează oriunde este acceptat un ID, și de obicei este exact ce vrei.

Ca să vezi ce este într-un snapshot înainte să descarci ceva:

```bash
restic ls latest
restic ls latest /home/me/Documents
```

Iar ca să localizezi un fișier când nu mai ții minte unde stătea sau care snapshot îl mai are:

```bash
restic find "quarterly-report.ods"
```

`restic find` caută în toate snapshot-urile și îți spune care dintre ele conțin potrivirea — cea mai rapidă cale de a răspunde la „când am avut ultima copie bună a acestui fișier?”.

### Restaurează tot

```bash
restic restore latest --target ~/restore
```

`--target` este directorul în care se scrie arborele; căile originale sunt recreate dedesubt. **Restaurează întotdeauna într-un director nou și gol**, nu peste datele live, până ești sigur de ce primești.

Ca să restaurezi un snapshot anume în loc de cel mai nou, dă-i ID-ul:

```bash
restic restore 79766175 --target ~/restore
```

### Restaurează un singur fișier sau folder

`--include` restrânge ce se scrie:

```bash
restic restore latest --target ~/restore --include /home/me/Documents/taxes
```

Poți de asemenea ancora snapshot-ul la un subfolder, ceea ce păstrează rezultatul mai plat în loc să recreeze toată calea sub target:

```bash
restic restore latest:/home/me/Documents --target ~/restore --include /taxes
```

`--exclude` funcționează la fel, dar invers, iar `--iinclude` / `--iexclude` sunt variantele care ignoră diferența între majuscule și minuscule.

Atenție că `--path` și `--host` fac altceva decât sugerează numele: ele **selectează care snapshot** este restaurat, nu care fișiere ies din el. Sunt utile când un repository conține mai multe mașini:

```bash
restic restore latest --host homeserver --target ~/restore
```

### Vezi în avans ce s-ar întâmpla

Pe restic 0.17.0 și mai nou, poți vedea ce ar face o restaurare fără să scrii nimic:

```bash
restic restore latest --target ~/restore --dry-run --verbose=2
```

Merită folosit înainte de orice restaurare pe loc, pentru că rularea de probă raportează care fișiere ar rămâne neatinse și care ar fi suprascrise. Dacă ai instalat pachetul 0.16.4 din Ubuntu, această opțiune nu există — este motivul principal pentru care să iei binarul upstream.

### Scoate un fișier fără o restaurare completă

Pentru un singur fișier, `dump` îl scrie direct la stdout:

```bash
restic dump latest /home/me/Documents/notes.md > notes.md
```

Funcționează și pentru directoare, ca flux de arhivă:

```bash
restic dump latest /home/me/Documents > documents.tar
restic dump -a zip latest /home/me/Documents > documents.zip
```

Este adesea cea mai rapidă cale când vrei doar să te uiți la o versiune mai veche a ceva.

### Navighează repository-ul ca un sistem de fișiere

```bash
mkdir ~/restic-mount
restic mount ~/restic-mount
```

Fiecare snapshot apare ca un arbore de directoare prin care poți naviga, pe care îl poți compara cu `diff` față de copia live sau din care poți copia fișiere individuale cu `cp`. Este read-only și are nevoie de FUSE, pe care Ubuntu îl are implicit. `Ctrl+C` demontează.

### Verifică periodic

Verificările de integritate și o restaurare de probă își au locul în calendar, aproximativ lunar:

```bash
restic check
restic check --read-data-subset=5%
restic restore latest --target /tmp/restore-test --include /home/me/Documents
```

`restic check` simplu validează structura și metadatele repository-ului. `--read-data-subset=5%` merge mai departe și descarcă și calculează efectiv hash-ul unui eșantion din date, ceea ce prinde corupția silențioasă din blocurile stocate. Verificarea completă cu `--read-data` înseamnă redescărcarea întregului repository, așa că un eșantion lunar este compromisul rezonabil.

## Depanare

**`Fatal: unable to open config file: <config/> does not exist`**

Calea repository-ului din script nu corespunde cu ce este în Dropbox, sau `restic init` nu a fost rulat niciodată. Verifică ambele:

```bash
rclone lsd dropbox:
rclone ls dropbox:restic-backups-homeserver --max-depth 1
```

A doua comandă ar trebui să listeze un fișier `config`. Dacă directorul lipsește complet, rulează `restic init`.

**`too_many_requests` — limitare de rată de la Dropbox**

Limitează rata de cereri a rclone. Orice opțiune rclone poate fi setată printr-o variabilă de mediu, cea mai curată metodă din scriptul de backup:

```bash
export RCLONE_TPSLIMIT=10
```

Alternativ, suprascrie argumentele pe care restic le transmite către rclone:

```bash
restic -o rclone.args="serve restic --stdio --tpslimit 10" snapshots
```

Ține minte că asta înlocuiește lista implicită de argumente — `serve restic --stdio --b2-hard-delete` — în loc să adauge la ea, deci păstrează `serve restic --stdio` pe poziție. `RCLONE_BWLIMIT` funcționează la fel, dacă ai nevoie să limitezi lățimea de bandă în loc de rata de cereri.

**`didn't find section in config file` sau alte erori de autentificare rclone**

Numele remote-ului din șirul repository-ului nu corespunde unui remote configurat:

```bash
rclone listremotes
```

Ar trebui să afișeze `dropbox:`. Reține că un systemd user service citește `~/.config/rclone/rclone.conf` ca utilizatorul tău, deci un remote configurat cu `sudo` nu va fi vizibil pentru timer.

**Timer-ul nu a rulat niciodată**

Verifică starea lui și dacă linger este activ:

```bash
systemctl --user list-timers
loginctl show-user $USER | grep Linger
journalctl --user -u restic-backup -e
```

Un timer care afișează `n/a` la următoarea rulare are de obicei o problemă de activare sau de linger, nu o expresie de calendar greșită — confirmă expresia separat cu `systemd-analyze calendar`.

## Documentație oficială

- [Documentația restic](https://restic.readthedocs.io/) și în special [capitolul despre restaurare](https://restic.readthedocs.io/en/stable/050_restore.html)
- [Pregătirea unui repository restic nou](https://restic.readthedocs.io/en/stable/030_preparing_a_new_repo.html), care acoperă backend-ul rclone și opțiunile lui
- [Backend-ul Dropbox din rclone](https://rclone.org/dropbox/) și [instalarea rclone](https://rclone.org/install/)
- `man systemd.timer` și `man systemd.time` pentru sintaxa completă de calendar
