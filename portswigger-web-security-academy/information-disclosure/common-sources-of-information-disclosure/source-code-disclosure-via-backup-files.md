---
description: >-
  https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-via-backup-files
icon: vial
---

# Source code disclosure via backup files

## Description

This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code.

## Solution

<figure><img src="../../../.gitbook/assets/image (22) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking source code page (CTRL+U) and discovering others pages there're not of interesting, so the idea is to try to guess a potential backup page (eg. git, backup).

In this case i was lucky, because the correct directory path was backup: [https://0a5400e703e5812285d9c2ef009700ba.web-security-academy.net/backup](https://0a5400e703e5812285d9c2ef009700ba.web-security-academy.net/backup)

<figure><img src="../../../.gitbook/assets/image (25) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

with the relative file .bak called: ProductTemplate.java.bak that contain a java package with relative DB information and the password/answer that we need to solve the lab.

<figure><img src="../../../.gitbook/assets/image (23) (1) (1) (1) (1).png" alt=""><figcaption><p><a href="https://0a5400e703e5812285d9c2ef009700ba.web-security-academy.net/backup/ProductTemplate.java.bak">https://0a5400e703e5812285d9c2ef009700ba.web-security-academy.net/backup/ProductTemplate.java.bak</a></p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (24) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
