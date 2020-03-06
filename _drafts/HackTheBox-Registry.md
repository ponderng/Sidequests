---
title: 'HackTheBox: Registry'
date: 2020-02-28
categories:
  - HackTheBox
tags: [docker, linux, nginx, restic, cms, webshell, ssh, reverse port forward, jtr, CVE-2019-9185, python, anti-csrf]
toc: true
toc_sticky: false
toc_label: "Table of Awesome Content"
toc_icon: "cog"
---

![Registry Box](/Sidequests/assets/registry/Registry-Logo.png)

| Name:             | Registry ![](/Sidequests/assets/icons/box-registry.png){:.img-av} |
|-------------------|--:|
| Release Date:     | 13 Jul 2019  |
| OS:               | Linux ![](/Sidequests/assets/icons/Linux.png){:.img-os} |
| Points:           | **Hard [40]**{:.diff-hard}  |
| Rated Difficulty: | ![](/Sidequests/assets/registry/registry-diff.png)  |
| Characteristics:  | ![](/Sidequests/assets/registry/registry-radar.png)  |
| Creator:          | thek ![](/Sidequests/assets/icons/user-thek.png){:.img-av} |

# About the box
Registry took me through several services that I've never used before, including one that I wanted to get to know better, Docker. The box begins with an online Docker registry available that's protected by easily guessable creds. After getting into the docker image, credentials are found that get SSH access for user. Using the SSH, enumerate the machine to find another service called "Bolt CMS" and figure out how to upload a webshell to get access to a second user. Then use another service "Restic Backup" to exfiltrate the root flag. Also, I explore a way to get root shell access.

# Initial Recon and Scans
## Port Scans

## Domains and Web Directories

