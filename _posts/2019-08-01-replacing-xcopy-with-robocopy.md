---
title: "Using robocopy to sync files"
excerpt: "This post describes switching from xcopy to robocopy to keep files and folders in sync"
date: 2019-08-01T08:00:30-04:00
categories:
  - blog
tags:
  - robocopy
---

I decided to update some scripts that relied on [xcopy][xcopy] to use [robocopy][robocopy] instead. Xcopy and robocopy are both command-line utilities that are regularly used to move around files. The description of the robocopy command is simply:

> Copies file data.

robocopy stands for Robust File Copy and is designed to be reliable and supports mirroring. The use-case I had at hand is to mirror folders with subfolders. This was previously done by using xcopy. However since xcopy isn't designed to mirror folders like robocopy there were some inconcistencies and that's the reason for this switch.

The xcopy command I'm about to replace is

```bash
xcopy /e /d /y <source> <dest>
```

The switches used here basically says to copy every folder and subfolder and all files that are newer than existing files in the destination. The /y switch suppresses a confirm-prompt when overwriting a destination file.

Going to robocopy I'm going to go for this command:

```bash
robocopy <source> <dest> /e /purge /r:5 /mt:32
```

`/e` and `/purge` together is almost the same as the `/mir`-switch that mirrors the folders, except for `/e /purge`:

> if the destination directory exists, the destination directory security settings are not overwritten.

And this is what I want in this case. Otherwise just use `/mir`. 

About the final switch `/mt` this tells robocopy to run this in multiple-threads. According to the [robocopy-docs][robocopy]:

> The /MT parameter applies to Windows Server 2008 R2 and Windows 7.

I'm testing this out on a later version of Windows anyway since the parameter is present. The default value when using only `/mt` is 8 threads, I'm going for 32 switch tells the command to (32 threads).

When trying out single-threaded without the `/mt` the elapsed time is 3 minutes and 19 seconds:

![robocopy-single-thread][robocopy-single-thread]

And multi-threaded with two cores the same operation gets down to 41 seconds:

![robocopy-multi-thread][robocopy-multi-thread]

The switch is still supported in later versions of Windows and apparently works as expected, so I advice you to try it out if you need it anyway.

[xcopy]:                  https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/xcopy
[robocopy]:               https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy
[robocopy-single-thread]: /assets/images/robocopy-single-thread.png
[robocopy-multi-thread]:  /assets/images/robocopy-multi-thread.png