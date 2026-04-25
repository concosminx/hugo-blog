---
date: '2026-04-25T10:00:00+02:00'
draft: false
title: 'Comenzi de bază WSL — Cheatsheet'
tags: ["wsl", "windows", "linux", "ubuntu", "cli"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "Un ghid practic cu cele mai utile comenzi WSL (Windows Subsystem for Linux) — de la instalare până la gestionarea distribuțiilor și montarea discurilor."
cover:
    image: wsl-basics.png
    alt: 'Comenzi de bază WSL'
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

## Introducere

**Windows Subsystem for Linux (WSL)** îți permite să rulezi un mediu Linux direct pe Windows, fără a folosi o mașină virtuală. Indiferent dacă instalezi prima distribuție Linux sau gestionezi mai multe, comanda `wsl` este instrumentul tău principal.

> **Notă:** Comenzile de mai jos funcționează în **PowerShell** sau **Command Prompt**. Dacă le rulezi dintr-un shell Bash / Linux, înlocuiește `wsl` cu `wsl.exe`.

---

## Instalare WSL

```powershell
wsl --install
```

Instalează WSL și distribuția implicită **Ubuntu**. Poți instala și o distribuție specifică:

```powershell
wsl --install --distribution <NumeDistributie>
```

Opțiuni frecvente de instalare:

| Opțiune | Descriere |
|---|---|
| `--distribution <Nume>` | Specifică distribuția Linux de instalat |
| `--no-launch` | Instalează fără a lansa automat distribuția |
| `--web-download` | Descarcă de pe internet în loc de Microsoft Store |
| `--location <Cale>` | Alege folderul de instalare |
| `--inbox` | Folosește componenta Windows în loc de Store |
| `--no-distribution` | Instalează doar motorul WSL, fără distribuție |

> **Sfat:** Pentru a lista distribuțiile valide, rulează `wsl --list --online`.

---

## Listează distribuțiile disponibile (online)

```powershell
wsl --list --online
# sau prescurtat:
wsl -l -o
```

Afișează toate distribuțiile Linux disponibile în Microsoft Store.

---

## Listează distribuțiile instalate

```powershell
wsl --list --verbose
# sau prescurtat:
wsl -l -v
```

Afișează toate distribuțiile instalate împreună cu **starea** lor (Running / Stopped) și **versiunea WSL** (1 sau 2).

Opțiuni suplimentare:

| Opțiune | Descriere |
|---|---|
| `--all` | Listează toate distribuțiile |
| `--running` | Listează doar distribuțiile active |
| `--quiet` | Afișează doar numele distribuțiilor |

---

## Setează versiunea WSL pentru o distribuție

```powershell
wsl --set-version <NumeDistributie> <1|2>
```

Comută o distribuție între WSL 1 și WSL 2. Exemplu:

```powershell
wsl --set-version Ubuntu 2
```

> **Atenție:** Comutarea versiunilor poate dura și poate eșua pentru proiecte mari. Fă o copie de rezervă a datelor înainte.

---

## Setează versiunea implicită WSL

```powershell
wsl --set-default-version <1|2>
```

Setează versiunea WSL implicită folosită pentru **noile** instalări de distribuții. WSL 2 este recomandat pe Windows 10 (build 18362+) și Windows 11.

---

## Setează distribuția implicită

```powershell
wsl --set-default <NumeDistributie>
```

Stabilește ce distribuție vor folosi comenzile `wsl` în mod implicit.

---

## Pornește WSL în directorul home

```powershell
wsl ~
```

Deschide WSL și te plasează direct în directorul home al utilizatorului Linux (`~`). Din interiorul shell-ului te poți întoarce oricând acasă cu:

```bash
cd ~
```

---

## Rulează o distribuție și un utilizator specific

```powershell
wsl --distribution <NumeDistributie> --user <NumeUtilizator>
```

Lansează o distribuție specifică ca un utilizator specific. Exemplu:

```powershell
wsl --distribution Debian --user root
```

> Rulează `whoami` în WSL pentru a verifica utilizatorul curent.

---

## Actualizează WSL

```powershell
wsl --update
```

Actualizează WSL la cea mai recentă versiune. Folosește `--web-download` pentru a descărca direct de pe GitHub:

```powershell
wsl --update --web-download
```

---

## Verifică starea WSL

```powershell
wsl --status
```

Afișează informații generale despre configurația WSL: tipul distribuției implicite, distribuția implicită și versiunea kernelului.

---

## Verifică versiunea WSL

```powershell
wsl --version
```

Afișează informații detaliate despre versiunea WSL și toate componentele sale.

---

## Ajutor

```powershell
wsl --help
```

Listează toate comenzile și opțiunile disponibile.

---

## Rulează ca un utilizator specific

```powershell
wsl --user <NumeUtilizator>
```

Lansează WSL ca utilizatorul specificat (trebuie să existe deja în distribuție).

---

## Schimbă utilizatorul implicit al unei distribuții

```powershell
<NumeDistributie> config --default-user <NumeUtilizator>
```

Exemplu — setează utilizatorul implicit Ubuntu la `johndoe`:

```powershell
ubuntu config --default-user johndoe
```

> **Notă:** Această comandă nu funcționează pentru distribuțiile importate. Pentru acestea, editează `/etc/wsl.conf`.

---

## Oprește toate instanțele WSL

```powershell
wsl --shutdown
```

Termină imediat **toate** distribuțiile active și mașina virtuală WSL 2. Util după modificarea fișierului `.wslconfig` sau la schimbarea limitelor de memorie.

---

## Oprește o distribuție specifică

```powershell
wsl --terminate <NumeDistributie>
```

Oprește distribuția specificată fără a o afecta pe celelalte.

---

## Identifică adresele IP

```powershell
# IP-ul distribuției Linux (VM WSL 2):
wsl hostname -I

# IP-ul Windows-ului văzut din WSL 2:
wsl -- ip route show | grep -i default | awk '{ print $3}'
```

---

## Exportă o distribuție

```powershell
wsl --export <NumeDistributie> <NumeFisier>
```

Salvează un snapshot al distribuției ca fișier `.tar` (util pentru backup sau migrare). Adaugă `--vhd` pentru a exporta ca fișier `.vhdx` (doar WSL 2).

---

## Importă o distribuție

```powershell
wsl --import <NumeDistributie> <LocatieInstalare> <NumeFisier>
```

Restaurează o distribuție dintr-un fișier `.tar`. Opțiuni:

| Opțiune | Descriere |
|---|---|
| `--vhd` | Importă dintr-un fișier `.vhdx` (doar WSL 2) |
| `--version <1\|2>` | Specifică versiunea WSL pentru distribuția importată |

---

## Importă o distribuție în loc (in-place)

```powershell
wsl --import-in-place <NumeDistributie> <NumeFisier>
```

Înregistrează un fișier `.vhdx` existent (trebuie să folosească sistemul de fișiere `ext4`) ca distribuție — fără a face o copie.

---

## Dezînregistrează / Dezinstalează o distribuție

```powershell
wsl --unregister <NumeDistributie>
```

Elimină distribuția din WSL. **Toate datele, setările și programele sunt șterse permanent.** Pentru a reinstala o copie curată, găsește distribuția în Microsoft Store.

---

## Montează un disc sau dispozitiv

```powershell
wsl --mount <CaleDisk>
```

Atașează și montează un disc fizic în toate distribuțiile WSL 2. Opțiuni frecvente:

| Opțiune | Descriere |
|---|---|
| `--vhd` | Tratează calea ca un disc virtual |
| `--name <Nume>` | Folosește un nume personalizat pentru punctul de montare |
| `--bare` | Atașează fără a monta |
| `--type <SistemFisiere>` | Tipul sistemului de fișiere (implicit: `ext4`) |
| `--partition <N>` | Indexul partiției (implicit: întreg discul) |
| `--options <OptiuniMontare>` | Opțiuni specifice sistemului de fișiere |

---

## Demontează un disc

```powershell
wsl --unmount <CaleDisk>
```

Demontează discul specificat. Fără `<CaleDisk>`, demontează **toate** discurile montate.

---

## Tabel de referință rapidă

| Comandă | Descriere |
|---|---|
| `wsl --install` | Instalează WSL + Ubuntu implicit |
| `wsl -l -o` | Listează distribuțiile disponibile online |
| `wsl -l -v` | Listează distribuțiile instalate cu stare |
| `wsl --set-version <distro> 2` | Comută o distribuție la WSL 2 |
| `wsl --set-default-version 2` | Setează WSL 2 implicit pentru instalări noi |
| `wsl --set-default <distro>` | Setează distribuția implicită |
| `wsl ~` | Deschide WSL în directorul home |
| `wsl --update` | Actualizează WSL |
| `wsl --status` | Afișează informații de configurare WSL |
| `wsl --shutdown` | Oprește toate distribuțiile active |
| `wsl --terminate <distro>` | Oprește o singură distribuție |
| `wsl --export <distro> <fisier>` | Backup distribuție |
| `wsl --import <distro> <cale> <fisier>` | Restaurare distribuție |
| `wsl --unregister <distro>` | Elimină permanent o distribuție |

---

## Referințe

- [Comenzi de bază WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)
- [Instalare WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Comparație WSL 1 și WSL 2](https://learn.microsoft.com/en-us/windows/wsl/compare-versions)
