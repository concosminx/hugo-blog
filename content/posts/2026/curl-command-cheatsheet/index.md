---
date: '2026-08-29T10:00:00+03:00'
draft: false
title: 'cURL Command Cheatsheet'
tags: ["cheatsheet", "cli"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "The curl flags worth memorising — sending data, inspecting headers, handling cookies and timeouts, and using exit codes to make curl behave in scripts."
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---

`curl` has over two hundred options, and you will use perhaps twenty of them. This is that twenty — the flags that come up when debugging an API, scripting a health check, or working out why a request behaves differently from the browser.

## Saving output

By default curl writes the response body to stdout. Two flags redirect it, and the difference between them is where the filename comes from:

```bash
curl -o page.html https://example.com
curl -O https://example.com/archive.tar.gz
```

`-o` takes the name you give it. `-O` reuses the remote filename — convenient for downloads, but it takes the name from the URL, so a URL ending in a path segment rather than a filename gives you something unhelpful.

Redirects are not followed unless you ask:

```bash
curl -L https://example.com
```

Without `-L` a 301 returns the redirect response itself, not the page. This is the single most common reason a curl request "returns nothing" when the browser loads the page fine.

## Sending requests

The method defaults to GET, or POST when you attach data. Set it explicitly with `-X`:

```bash
curl -X DELETE https://api.example.com/items/42
```

Data goes in with `-d`:

```bash
curl -d 'name=value&other=thing' https://api.example.com/submit
curl -d @payload.json https://api.example.com/submit
```

Prefixing with `@` reads the body from a file. Note that `-d` processes that file — newlines and carriage returns are stripped — which is fine for JSON but corrupts anything binary. When the bytes must arrive exactly as they are, use the binary form:

```bash
curl --data-binary @image.png https://api.example.com/upload
```

Custom headers are repeatable, one flag per header:

```bash
curl -H 'Content-Type: application/json' -H 'Authorization: Bearer token' \
     -d @payload.json https://api.example.com/submit
```

The user agent has its own shorthand, since overriding it is common enough to deserve one:

```bash
curl -A 'Mozilla/5.0' https://example.com
```

## Inspecting the exchange

Three levels of detail, depending on how much you need to see.

`-i` includes the response headers above the body:

```bash
curl -i https://example.com
```

`-I` sends a HEAD request and prints only the headers, transferring no body at all — the right choice for checking a status code or a redirect target:

```bash
curl -I https://example.com
```

`-v` shows the whole conversation, including the request curl actually sent:

```bash
curl -v https://example.com
```

In verbose output, `>` marks data sent by curl and `<` marks data received. That distinction is what makes `-v` worth reaching for: most "the server is wrong" bugs turn out to be a header you did not send.

The opposite direction, for scripts:

```bash
curl -s https://example.com
curl -sS https://example.com
```

`-s` suppresses the progress meter and error messages. That silences real failures too, so pair it with `-S`, which puts errors back. `-sS` is the combination you want in a cron job — quiet when it works, loud when it doesn't.

## Cookies

`-b` sends cookies and `-c` stores them. Used together they give you a session across invocations:

```bash
curl -c jar.txt -d 'user=me&pass=secret' https://example.com/login
curl -b jar.txt https://example.com/account
```

`-b` also accepts cookies inline, which is quicker when you only need one or two:

```bash
curl -b 'session=abc123; theme=dark' https://example.com
```

## Authentication and proxies

HTTP authentication takes a colon-separated pair:

```bash
curl -u username:password https://example.com
```

Omitting the password (`-u username`) makes curl prompt for it, which keeps the credential out of your shell history and out of the process list — worth the extra keystroke on a shared machine.

Routing through a proxy:

```bash
curl -x 'http://proxy:2080' https://example.com
```

## Timeouts and retries

Unbounded requests are how a script hangs forever. Two different limits, and they are not interchangeable:

```bash
curl --connect-timeout 5 -m 30 https://example.com
```

`--connect-timeout` caps only the time spent establishing the connection; `-m` caps the whole operation including the transfer. A large download needs a generous `-m` but still benefits from a tight `--connect-timeout`, so setting both is usually right.

For transient failures, curl can retry on its own:

```bash
curl --retry 3 --retry-delay 2 https://example.com
```

## Compression

Ask the server to compress the response:

```bash
curl --compressed https://example.com
```

curl advertises the encodings it was built with — typically gzip, deflate, br and zstd — and decompresses the response transparently, so what reaches stdout is the same content either way. The flag is `--compressed`, not `--compression`; the latter is not a curl option and will fail outright.

## TLS

Certificate verification failures abort the request. To proceed anyway:

```bash
curl -k https://self-signed.example.com
```

`-k` (long form `--insecure`) is for testing against staging boxes and self-signed certificates. It disables verification entirely, which means it also disables the protection you were relying on, so it does not belong in anything that runs unattended.

## Scripting with curl

Two features turn curl from an interactive tool into a usable script component.

The first is `-w`, which prints a chosen format string after a successful transfer:

```bash
curl -s -w '%{remote_ip} %{time_total} %{http_code}\n' -o /dev/null https://example.com
```

Sending the body to `/dev/null` and writing only the metrics gives you a one-line health check. Useful variables include:

| Variable | Meaning |
| --- | --- |
| `%{http_code}` | The response status code |
| `%{time_total}` | Total transfer time in seconds |
| `%{time_connect}` | Time until the connection was established |
| `%{time_starttransfer}` | Time to the first byte |
| `%{remote_ip}` | The IP curl actually connected to |
| `%{size_download}` | Bytes downloaded |
| `%{json}` | Every available variable, as a JSON object |

The second is exit codes. curl exits 0 for a *completed transfer*, which includes a 404 or a 500 — the request worked, the server just said no. To make HTTP errors fail the command:

```bash
curl -f https://example.com/missing
```

`-f` (`--fail`) suppresses the error body and returns a non-zero exit code on HTTP errors, which is what you want when the next line of the script depends on success.

The exit codes worth recognising:

| Code | Meaning |
| --- | --- |
| 6 | Could not resolve host |
| 7 | Failed to connect to host or proxy |
| 28 | Operation timed out |
| 55 | Failed sending network data |
| 56 | Failed receiving network data |

The distinction between 6 and 7 is the one that saves time: 6 is DNS, 7 is the connection itself. Reaching for `curl -v` before knowing which of the two you have is wasted effort.

## A note on Windows

`curl.exe` ships with Windows 10 and 11, but in **Windows PowerShell 5.1** the name `curl` is an alias for `Invoke-WebRequest`, a different command with entirely different arguments. A working command line fails with confusing parameter errors as a result. Call the executable by its full name:

```powershell
curl.exe -I https://example.com
```

PowerShell 7 removed the alias, so `curl` there is the real thing. In cmd.exe, Git Bash and WSL the name has always resolved to curl itself.

## Further reading

`man curl` is long but genuinely worth skimming once. The [official documentation](https://curl.se/docs/) covers the options in full, and `curl --help all` lists every flag in the version you actually have installed — which is the fastest way to check whether an option you read about somewhere exists in your build.
