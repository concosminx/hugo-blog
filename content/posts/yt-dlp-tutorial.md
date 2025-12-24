---
date: '2025-12-23T14:00:00+02:00'
draft: false
title: 'yt-dlp: The YouTube Downloader Tutorial'
tags: ["youtube", "download", "video", "audio", "cli", "python"]
categories: ["tech"]
showToc: true
TocOpen: false
author: "Me"
cover:
    image: img/yt-dlp-cover.jpg
    alt: 'yt-dlp command line interface'
    #caption: 'yt-dlp command line interface'
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
#hidemeta: false
#comments: false
description: "Learn how to install and use yt-dlp, the powerful command-line tool for downloading videos and audio from YouTube"
#canonicalURL: "https://canonical.url/to/page"
#disableHLJS: true # to disable highlightjs
#disableShare: false
#disableHLJS: false
#hideSummary: false
#searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
#ShowRssButtonInSectionTermList: true
#UseHugoToc: true
---

## Why this tutorial?

While there are numerous online solutions and dedicated software (many paid) for downloading from YouTube, I was looking for a solution that gives me complete control over the process. yt-dlp seems like the perfect answer - free, open-source, and incredibly flexible. The less amusing part? Flexibility comes bundled with an encyclopedia of options that can make you feel like an airplane pilot trying to understand the cockpit dashboard.

🛩️ This tutorial exists to save you a few hours of "wait, what does this flag do again?"


## What is yt-dlp?

yt-dlp is a Python-based command-line program that allows you to download videos from YouTube, Vimeo, Twitter, TikTok, and many other video hosting platforms. It supports various formats, quality options, and advanced features like playlist downloads and live stream recording.

## Installation

### Windows

#### Download executable
1. Go to the [yt-dlp releases page](https://github.com/yt-dlp/yt-dlp/releases)
2. Download `yt-dlp.exe`
3. Add the executable to your PATH or run it from the download directory

> **Note:** Alternative installation methods are available including Python pip, Chocolatey, and Windows Package Manager (winget). 

## Installing FFmpeg (Recommended)

For advanced features like audio conversion and video post-processing, you'll need **FFmpeg**. While yt-dlp can download videos without it, FFmpeg significantly enhances its capabilities.

### Installing FFmpeg on Windows

#### Download Pre-compiled Binaries (Easiest)
1. Go to [FFmpeg Windows builds](https://github.com/yt-dlp/FFmpeg-Builds/wiki/Latest)
2. Download the `win64-gpl` variant
3. Extract the ZIP file to `C:\ffmpeg`
4. Add `C:\ffmpeg\bin` to your system PATH:
   - Press `Win + R`, type `sysdm.cpl`, press Enter
   - Go to "Advanced" tab → "Environment Variables"
   - Under "System Variables", find and select "Path" → "Edit"
   - Click "New" and add: `C:\ffmpeg\bin`
   - Click "OK" on all dialogs


### Verify Installation
Open Command Prompt and run:
```bash
ffmpeg -version
```

If installed correctly, you'll see FFmpeg version information.


> **⚠️ Disclaimer:** The installation instructions provided focus primarily on Windows. For macOS, Linux, and other operating systems, please refer to the [official yt-dlp documentation](https://github.com/yt-dlp/yt-dlp#installation) for platform-specific installation methods and requirements.


## Basic Usage


### Download YouTube video as MP3
```bash
yt-dlp -x --audio-format mp3 "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Download a single video
```bash
yt-dlp "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Download audio only (best quality)
```bash
yt-dlp -f bestaudio "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Download video in specific quality
```bash
yt-dlp -f "best[height<=720]" "https://www.youtube.com/watch?v=VIDEO_ID"
```

## Advanced Features

### Download entire playlists
```bash
yt-dlp "https://www.youtube.com/playlist?list=PLAYLIST_ID"
```

> **Note:** The playlist must be public. 

### Extract audio and convert to MP3
```bash
yt-dlp -x --audio-format mp3 "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Download multiple YouTube Videos as MP3s
```bash
yt-dlp -x --audio-format mp3 -i PLAYLIST_ID
```

### List available formats
```bash
yt-dlp -F "https://www.youtube.com/watch?v=VIDEO_ID"
```

> **Note:** After that, take a look at the [format selection section](https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#format-selection).

### Download a YouTube video in best quality
```bash
yt-dlp -f "best" "https://www.youtube.com/watch?v=VIDEO_ID"
```

> **Note:** This selects the best pre-merged format which is often not the best option. To let yt-dlp download and merge the best available formats, simply do not pass any format selection.

### Batch download from file
```bash
# Create a file with URLs (one per line)
yt-dlp -a urls.txt
```

### Download a file in mp4 format from playlist without downloading the full playlist 
```bash
yt-dlp -f mp4 --no-playlist "SOME_VIDEO_FROM_PLAYLIST_ID"
```

## Troubleshooting

### Update yt-dlp regularly
YouTube frequently changes its structure, so keep yt-dlp updated:
```bash
yt-dlp -U
```

### Common issues
- **"Video unavailable"**: The video might be region-locked or private
- **Slow downloads**: Try using `--limit-rate` to avoid throttling
- **Format not available**: Use `-F` to list available formats first

## References

- [Official yt-dlp Repository](https://github.com/yt-dlp/yt-dlp) - Complete documentation, latest releases, and source code
- [Simple yt-dlp Guide - Reddit Discussion](https://www.reddit.com/r/youtubedl/comments/qzqzaz/can_someone_please_post_a_simple_guide_on_making/) - Community discussion with practical tips and examples