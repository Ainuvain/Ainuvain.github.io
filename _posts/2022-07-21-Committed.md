---
layout: post
title:  "Committed"
date:   2022-07-21 16:30:02 +0200
author: Ainut
categories: writeups
tags: git security tryhackme
---

Committed is a challenge room in TryHackMe about accidental git commits. Link to the challenge [tryhackme.com/room/committed](https://tryhackme.com/room/committed)

**Contents**
* list
{:toc}
<br>

## Extracting commits

We are provided a zip file of git repository and information that there has been a commit with sensitive information. 

Starting with unzipping the `committed.zip` and then using extractor script from <a href="https://github.com/internetwache/GitTools/blob/master/Extractor/" target="_blank">GitTools</a> to restore the commits in `.git` 

`./extractor.sh . commit`

We got nine commits in total: 

[![commits.png](/assets/img/committed/commits.png)](/assets/img/committed/commits.png)

## Finding the sensitive information

We can use this one-liner to print every commit-meta.txt 

`separator="======================================="; for i in $(ls); do printf "\n\n$separator\n\033[4;1m$i\033[0m\n$(cat $i/commit-meta.txt)\n"; done; printf "\n\n$separator\n\n\n"`

The commit with a message *oops* seems related to issue, but it is just the fix for the mistake and we need to actually look into its parent

[![oops.png](/assets/img/committed/oops.png)](/assets/img/committed/oops.png)

Looking into the files in the `DB check` commit we can find the cause of oopsie in `main.py`

[![flag.png](/assets/img/committed/flag.png)](/assets/img/committed/flag.png)

So what do we learn: never make a commit with sensitive information.
