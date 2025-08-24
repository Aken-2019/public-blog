---
title: "[ETL] A Quick Way to Check If Files Are Landed Completely"
date: 2023-01-15T19:07:56+08:00
draft: false
---

## Introduction

When working for ETL projects, I find there is a need to check if files are landed (or downloaded) completely for next-step processing. We may suffer data lost if we process the files immediately when they are still being transferred. The ideal case is to negotiate a signal with your up-steam. The signal will be shown when all files are finished transferring. In this case, you don't need to perform an landing check. However, the real world is far from ideal. In this article, I will a quick ways to check file landing.

## Landing Check without Checksum
If you don't have checksums, such as sha1sum or md5sum, from your upstream, checking file size and last-modified date is your best bet. If the files are still landing, their sizes and timestamps will continually increase. You can use `ls -l --full-time` to get this kind info in great precision. It will list all the files in the working directory with  timestamps to nano seconds. Below is an example output:
```bash
total 72
-rw-r--r-- 1 abc abc 5561 2023-01-15 19:05:42.737438710 +0800 airflow-best-practices.md
-rw-r--r-- 1 abc abc  894 2023-01-15 19:05:42.737438710 +0800 a-life-on-our-planet.md
-rw-r--r-- 1 abc abc 2768 2023-01-15 19:05:42.737438710 +0800 books-2022-oct-16.md
-rw-r--r-- 1 abc abc 9910 2023-01-15 19:05:42.737438710 +0800 data-management-at-scale.md
-rw-r--r-- 1 abc abc  988 2023-01-15 19:50:15.437798695 +0800 etl-quick-ways-to-check-file-landing.md
-rw-r--r-- 1 abc abc 3403 2023-01-15 19:05:42.737438710 +0800 fcwkxb-and-wlsmsyzr.md
-rw-r--r-- 1 abc abc 3018 2023-01-15 19:05:42.737438710 +0800 la-fabrique-du-consommateur.md
-rw-r--r-- 1 abc abc   76 2023-01-15 19:05:42.737438710 +0800 my-first-post.md
-rw-r--r-- 1 abc abc 1647 2023-01-15 19:05:42.737438710 +0800 principles-of-risk-management-and-insurance.md
-rw-r--r-- 1 abc abc 1315 2023-01-15 19:05:42.737438710 +0800 react-learning-path.md
-rw-r--r-- 1 abc abc 5738 2023-01-15 19:05:42.737438710 +0800 revisit-iterator-in-depth.md
-rw-r--r-- 1 abc abc 2404 2023-01-15 19:05:42.737438710 +0800 the-devops-handbook.md
-rw-r--r-- 1 abc abc  703 2023-01-15 19:05:42.737438710 +0800 tumultuous-times.md
-rw-r--r-- 1 abc abc 3493 2023-01-15 19:05:42.737438710 +0800 what-is-education.md
```

File sizes are listed in the 5th column in bytes (*3493* in the last row for example) and timestamps in nano seconds (*19:05:42.737438710* for example) with the your timezone (*+080* for example). You can save the result to a bash variable, wait for some time, and do this again. You will be able to tell if the files are being changed by comparing the results. Below is a complete snippets:

```bash
fileinfo_snapshot_1=$(ls -l --full-time)
sleep 10
fileinfo_snapshot_2=$(ls -l --full-time)

if [ "${fileinfo_snapshot_1}" == "${fileinfo_snapshot_2}" ]; then
    echo "The files are not changed in 10 seconds. They should be landed completely."
else
    echo "File changes are detected. They are still being transferred."
fi
```

## Limitations
This solution will work in most cases. However, if there is some network outrage and the file landing is disrupted for a long time (longer than 10 seconds for the snippet above), you may still experience data lost. You can increase the waiting interval to reduce this kind of risk. IMO, ten-second waiting is good enough for stable network connection.



