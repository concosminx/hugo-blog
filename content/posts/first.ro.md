---
date: '2025-01-03T10:55:00+02:00'
draft: false
title: 'WinSCP - sudo dupa autentificare'
taguri: ["ssh", "ftp", "sftp", "linux", "ubuntu"]
categorii: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/pexels-harold-vasquez-853421-2653362.jpg
    alt: 'Cod pe tema IDE inchisa la culoare'
    #caption: 'Code on black IDE theme'
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
#hidemeta: false
#comments: false
#description: "Desc Text."
#canonicalURL: "https://canonical.url/to/page"
#disableHLJS: true # to disable highlightjs
#disableShare: false
#disableHLJS: false
#hideSummary: false
#searchHidden: true
#ShowReadingTime: true
#ShowBreadCrumbs: true
#ShowPostNavLinks: true
#ShowWordCount: true
#ShowRssButtonInSectionTermList: true
#UseHugoToc: true
---

Recent, a trebuit sa incarc cateva foldere pe un server Linux (Oracle Cloud), dar permisiunile utilizatorului pe care le aveam nu imi permiteau sa creez folderele necesare. In cautarea unei solutii, am dat peste aceasta posibilitate.  

In unele cazuri (cu servere Unix/Linux), poti folosi comanda `sudo` imediat dupa autentificare pentru a schimba utilizatorul, inainte ca sesiunea de transfer de fisiere sa inceapa.  

Protocoalele *SFTP* si *SCP* permit acest lucru, dar metoda efectiva depinde de platforma.  

Cu protocolul *SFTP*, poti folosi optiunea SFTP server de pe pagina **SFTP** din dialogul *Advanced Site Settings* pentru a executa binarul SFTP sub un alt utilizator.  
- Cu serverul *OpenSSH*, poti specifica: `sudo /bin/sftp-server`. Retine ca binarul serverului SFTP poate fi localizat in alta parte (de exemplu, in `/usr/lib/sftp-server`, `/usr/lib/openssh/sftp-server` sau `/usr/libexec/openssh/sftp-server`).  
- Cu protocolul *SCP*, poti specifica urmatoarea comanda ca shell personalizat pe pagina **SCP/Shell** din dialogul *Advanced Site Settings*: `sudo -s`.  

Totusi, nu vei putea introduce o parola pentru comanda `su` (vezi limitarile executiei comenzilor remote). Asadar, vei putea face cele de mai sus doar daca ai permisiunea de a executa `sudo su` fara a ti se cere o parola.


[sursa](https://winscp.net/eng/docs/faq_su)