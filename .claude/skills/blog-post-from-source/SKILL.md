---
name: blog-post-from-source
description: Turn a source document in the blog's source/ directory (PDF, Markdown, text, Word) into a complete bilingual Hugo post — an English index.md and a Romanian index.ro.md — with front matter matching this blog's conventions and tags chosen by the user. Use this whenever the user asks for a new blog post, a post based on a document/cheatsheet/PDF/guide, mentions writing, drafting or generating a post for this blog, or points at a file in source/ — even when they don't name this skill or say "bilingual".
---

# Blog post from a source document

This blog publishes every post twice: `index.md` in English and `index.ro.md` in Romanian. Your job is to read a source document, turn it into a genuine post rather than a transcription, and produce both language files so they stay in sync from the start. Retrofitting the Romanian version later is where drift creeps in.

Work through the steps in order. Three of them carry most of the risk: Step 2, because a source document is a claim and not yet a fact; Step 4, because tags are the user's call; and Step 7, because the build fails silently in two specific ways.

## Step 1 — Locate and read the source

Source documents live in `source/`, which is untracked. List it:

```bash
ls -la source/
```

If it's empty, ask the user for a path rather than guessing.

If several documents are there, ask whether they feed **one post** or **separate posts** — don't assume either. Two cheatsheets on the same tool are usually one post; a Kafka guide and a WSL reference are clearly two.

**Leave the source file where it is.** Don't move, rename or delete it — the user cleans up `source/` themselves, and a post often needs a second pass against the original.

Extraction depends on the format:

| Format | How to read it |
| --- | --- |
| `.md`, `.txt` | Read directly |
| `.pdf` | `pdftotext -layout <file> <scratch>/extracted.txt`, then Read that |
| `.docx` | Use the `docx` skill |

For PDFs, `-layout` matters: cheatsheets and reference cards are usually multi-column, and without it the columns interleave into nonsense. Extract to a scratch file rather than piping to stdout so you can re-read sections while drafting.

On this machine `pdftotext` (poppler) is on PATH, but `pdftoppm` is not installed and neither is `pypdf` nor `PyMuPDF`. So the Read tool's own PDF rendering will fail — go straight to `pdftotext`.

Read the **whole** extracted text before writing anything. A cheatsheet's structure — its sections and their order — is usually the right skeleton for the post, and you can only see that from the full document.

### When several documents feed one post

Overlapping sources are the normal case, not the exception — two references on the same tool will share most of their basics and differ only at the edges. Merge them into one post; don't walk through them one at a time.

Read every document first, then build a **single outline** covering the union of what they say, and write against that outline rather than against any one source. Working document-by-document is what produces a post that explains `install` in section 2 and again in section 5 — each pass feels new while you're writing it, and the repetition is only obvious to the reader.

The rule is that **each fact, command or concept appears exactly once**, in the section where it best belongs. Concretely:

- Where documents agree, state it once. Don't note that multiple sources cover it — that's a fact about your inputs, not about the topic.
- Where one document goes deeper, use the fuller treatment and drop the thinner one entirely rather than including both.
- Where documents genuinely differ, that's usually version drift rather than disagreement, and it's a Step 2 question. Verify which is current, write that, and mention the older behaviour only when a reader on an older system would otherwise be misled.
- Where a command appears in several contexts, put it in the most useful one and cross-reference with a relative link instead of restating it.

The finished post should read as though it came from one source. A reader should not be able to tell how many documents you had, or where one ended and the next began.

## Step 2 — Fact-check the material

A source document is a starting point, not an authority. Reference cards go stale quietly: flags get renamed, subcommands get deprecated, tools get replaced wholesale. Anything you publish is the blog's claim now, not the document's, so verify before you assert.

Check the source's own age first — a copyright line, version numbers, the distro or release it targets. That tells you how suspicious to be. A card written for RHEL 7 in 2014 needs every command checked; a document from last month needs spot checks.

Then verify against **primary sources** — official project documentation, the actual man page, upstream release notes. Use `WebSearch` and `WebFetch`. Vendor docs and man pages beat blog posts and Stack Overflow answers, which carry the same staleness problem as the document you're checking.

Concentrate on what actually breaks:

- **Commands and subcommands** — does it still exist, under that name?
- **Flags and options** — still spelled that way, still doing that?
- **Replacement and deprecation** — has the whole tool been superseded? This is the highest-value check, because it reframes the entire post rather than one line.
- **Version-specific behaviour** — the source often states something true only of the release it was written for.

Three ways to handle what you find:

1. **Confirmed** — write it plainly.
2. **Changed** — write what is true today, and say so where the difference matters to a reader who might be on an older system. Silently modernising a command loses useful information; noting "on RHEL 8+ this is `dnf`" keeps it.
3. **Unverifiable** — leave it out, or state the uncertainty explicitly. Never launder a claim you couldn't confirm into confident prose. Dropping a line costs nothing; a wrong command costs the reader an hour.

Keep a short list of what you corrected and what you couldn't confirm — the user needs it in Step 8.

Don't let this become a research project. You're checking that the material is sound, not writing a literature review. When the source is recent and the commands are stable, a few targeted lookups are enough.

## Step 3 — Plan the post

Pick a slug from the topic: lowercase, hyphenated, descriptive (`yum-command-cheatsheet`, `wsl-basic-commands`). The bundle path is:

```
content/posts/<year>/<slug>/index.md
content/posts/<year>/<slug>/index.ro.md
```

`<year>` is the current year. Both language files live in the same directory — this is a Hugo page bundle, so any images sit beside them and are referenced by bare filename.

**The date must be in the past.** `hugo.yaml` sets `buildFuture: false`, so a post dated even an hour ahead is silently dropped from the build — no warning, and the URL just 404s. Check the current time and back-date slightly:

```bash
date "+%Y-%m-%dT%H:%M:%S%z"
```

Then write it in the offset form the other posts use, e.g. `'2026-08-29T09:00:00+03:00'`.

## Step 4 — Ask the user for tags

Tags are the user's decision, not yours. Guessing fragments the taxonomy — a post tagged `sysadmin` when every neighbour says `linux` creates a tag page with one entry on it.

Start from what the blog already uses, so you propose real tags rather than inventing near-duplicates:

```bash
grep -h "^tags:" content/posts/*/*/index.md | sed 's/^tags: //' | tr -d '[]"' | tr ',' '\n' | sed 's/^ *//' | sort | uniq -c | sort -rn
```

Then ask with `AskUserQuestion`, `multiSelect: true`, offering the strongest candidates for this document — existing tags first, and a new one only when nothing fits. Say in the option descriptions that "Other" takes a comma-separated list, since the tool caps you at four options and the user may want a different set entirely.

Take the answer literally. When the user says "only the cheatsheet tag", the front matter is `tags: ["cheatsheet"]` — don't append the topic tags you had in mind.

`categories` is a separate field and is `["tech"]` on every post so far; carry that over without asking.

## Step 5 — Write the English post

Write `index.md` using this front matter:

```yaml
---
date: '<past timestamp with offset>'
draft: false
title: '<Title Case Title>'
tags: [<what the user chose>]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
description: "<one sentence, used for SEO and list summaries>"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
---
```

`draft: false` is the project rule in CLAUDE.md — posts are written to be published rather than staged. This means the post goes live on the next push to `main`, so the fact-check in Step 2 is the only thing standing between a stale source and a published claim. Treat it accordingly.

Omit the `cover:` block. Most posts have one, but you can't produce the artwork, and a `cover.image` pointing at a file that doesn't exist renders a broken image. Flag it in your final summary instead (Step 8).

On the body:

- **Never open with a `#` heading.** PaperMod renders the `title` field as the page's `<h1>`; a second one is a duplicate heading. Sections start at `##`.
- **Write prose, don't dump tables.** The point of a post is the connective tissue a reference card lacks — when you'd reach for a command, what the flags actually do, which of three similar subcommands to pick. Look at `content/posts/2026/linux-devops-cheatsheet/index.md` for the house voice: a short orienting sentence, the command in a fenced block, then a line or two on why it matters.
- Tables earn their place for genuinely flat lists — option flags, a command inventory — where prose would be padding.
- Fenced code blocks always carry a language (` ```bash `).
- **Say each thing once.** Reference cards repeat themselves by design — the same command turns up under two headings, and aliases like `remove` and `erase` get separate entries. A post shouldn't inherit that. Cover it in the place it belongs, note the aliases inline, and link internally rather than restating. This matters most when several documents fed the post; see Step 1.
- **Never mention the source document.** No "based on Red Hat's cheat sheet", no vendor attribution, no "the original guide says", no filename. The source is raw material you worked from — the post is the blog's own writing and should read that way. A reader has no access to the document and no reason to care that it existed.
- Fold in what you learned in Step 2. A note that `yum` is a symlink to `dnf` on RHEL 8+ belongs in the post as plain fact — stated directly, not as a correction to something the reader never saw.
- General pointers outward are fine and often useful: `man yum`, a link to official project documentation. The rule is about not citing *your* source, not about refusing to help the reader go further.
- Relative links for anything internal.

## Step 6 — Write the Romanian post

`index.ro.md` is a real translation, not a second draft — the two files should teach the same thing in the same order, so a reader switching languages lands somewhere recognisable.

Copy these fields across **unchanged**: `date`, `draft`, `tags`, `categories`, and `cover.image` if one exists. Tags stay in English on both files — that's what keeps `/tags/` and `/taguri/` pointing at the same posts.

Translate `title`, `description`, `cover.alt`, and the body.

The Romanian in this blog reads like it was written in Romanian, not run through a translator. Concretely:

- **Full diacritics**: ă, â, î, ș, ț. Not optional.
- **Imperative second-person singular** for instructions: "Conectează-te la server", "Creează un utilizator nou".
- **Keep technical terms in English** and attach Romanian articles with a hyphen — `link-urile`, `thread dump-urilor`, `imagini Docker`. Translating "thread dump" into Romanian would be less clear to the audience, not more.
- **Never translate** commands, flags, file paths, package names, code, or output. Only the prose around them.
- Translate headings idiomatically: "Prerequisites" → "Cerințe preliminare", "Step 1 — Connect to the server" → "Pasul 1 — Conectează-te la server".

`content/posts/2026/create-sudo-user-ubuntu/index.ro.md` and `content/posts/2026/rustfs-docker-install/index.ro.md` are good models.

## Step 7 — Check the build

Both ways this can fail are silent — Hugo exits 0 and simply omits the page — so checking the exit code proves nothing. Build to a throwaway directory and confirm the pages exist:

```bash
hugo --quiet --buildDrafts --destination <scratch>/blogcheck
ls <scratch>/blogcheck/posts/<year>/<slug>/          # English
ls <scratch>/blogcheck/ro/posts/<year>/<slug>/       # Romanian
```

Note the Romanian version builds under `ro/` — checking only the English path would miss a broken `.ro.md`. Use your session's scratchpad directory for `<scratch>` rather than the repo's `public/`, which may hold a build the user is currently serving.

`--buildDrafts` is belt-and-braces: posts are `draft: false` now, but it also catches a post that picked up `draft: true` from the archetype.

If a directory is missing, the cause is almost always one of:

1. **The date is in the future** — `buildFuture: false` dropped it. Re-check against `date` and back-date.
2. **`draft: true` slipped in** — likely from `hugo new`, whose archetype still seeds it. The page then builds only under `--buildDrafts` and vanishes from the real site.

Fix and rebuild before reporting success.

## Step 8 — Report back

Tell the user:

- The two file paths you created.
- The tags you applied, so they can see their choice landed.
- **What the fact-check turned up**: anything in the source that was wrong or outdated and how you handled it, plus anything you could not confirm and therefore dropped or hedged. This is the part the user cannot check by reading the post — the corrections are invisible once folded into the prose, and a claim you quietly omitted leaves no trace at all. Keep it to a few lines; if the source held up, say that.
- That the post is `draft: false` and will go live on the next push to `main`. If a `hugo server` is already running it picks the post up automatically; a new one needs the running server stopped first, or port 1313 is taken.
- That there's no cover image yet, with the exact lines to add once they have one:

  ```yaml
  cover:
      image: <filename>.png
      alt: '<alt text>'
  ```

  and the reminder that `alt` should be translated in the `.ro.md` while `image` stays identical.

- Anything you added beyond the source, or judgement calls worth a second opinion.

## Scope

This skill covers creating a new post from a document. Editing an existing post, adding a translation to a post that already has only English, or changing tags across the archive are ordinary edits — just make them, keeping the conventions above in mind.
