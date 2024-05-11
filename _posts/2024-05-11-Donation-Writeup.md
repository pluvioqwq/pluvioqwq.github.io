---
title: Donation Writeup
date: 2024-05-11
categories:
  - Writeups
tags:
  - UMDCTF
---
# Intro

- Recently, I have participated in UMDCTF as member of F1ag dot txt, together with 1 other teammate. Due to time constraints, we only managed to solve few of them

# 0x01

By accessing the website, we can see a donation webpage.

# 0x02

We firstly register an account, and we can see that our initial amount is 1000.

# 0x03

By accessing the donation page, we intercept the packet by using burp first.

We can see that there are two post parameter: user and currency. We then found that we can actually donate negative amount (which means add the amount to ourselves). By doing such, when we return to the Profile Page, we can see the flag

![[Pasted image 20240427231412.png]]







