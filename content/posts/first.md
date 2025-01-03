---
date: '2025-01-03T10:55:00+02:00'
draft: false
title: 'WinSCP - sudo after login'
tags: ["ssh", "ftp", "sftp", "linux", "ubuntu"]
categories: ["tech"]
showToc: false
TocOpen: false
author: "Me"
cover:
    image: img/pexels-harold-vasquez-853421-2653362.jpg
    alt: 'Code on black IDE theme'
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

Recently, I had to upload some folders to a Linux server (Oracle Cloud), but the user permissions I had did not allow me to create the necessary folders. While searching for a solution, I came across this possibility.

In some cases (with Unix/Linux server) you may be able to use `sudo` command straight after login to change a user, before file transfer session starts. 

The *SFTP* and *SCP* protocols allow for this, but the actual method is platform dependent.

With SFTP protocol, you can use SFTP server option on SFTP page of Advanced Site Settings dialog to execute SFTP binary under a different user. 
- With OpenSSH server, you can specify: `sudo /bin/sftp-server`. Note that SFTP server binary may be located elsewhere (e.g. in `/usr/lib/sftp-server`, `/usr/lib/openssh/sftp-server` or `/usr/libexec/openssh/sftp-server`).
- With SCP protocol, you can specify the following command as custom shell on the SCP/Shell page of Advanced Site Settings dialog: `sudo -s`

However you will not be able to provide a password for su (see remote command execution limitations). So you may be able to do the above only if you are allowed to do `sudo su` without being prompted with password.

[source](https://winscp.net/eng/docs/faq_su)