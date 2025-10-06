---
title: MetaCTF August 2025 Writeups
categories:
  - MetaCTF Writeups
tags:
  - CTF
  - Webapp
  - Injection
classes: wide
---
## Table of Contents
 - [Overview](#overview)
 - [Challenges](#challenges)
    - [1. Baby Something (VERY Simple)](#1-baby-something)
    - [2. Super Quick Logic Invitational](#2-super-quick-logic-invitational)
 - [Conclusion](#conclusion)

## Overview

Hey there! In the previous post, I had mentioned that I was going to try and post a writeup once a month until August. As you can see, that didn't really pane out... but we're back! I'm behind quite a few months in regards to the monthly MetaCTF events, but I recently went back and attempted the challenges posted in August and was able to solve two: one forensic challenge that was quite simple, the other being a very interesting injection exploit! 

In this post, I will be covering my thought process as to how I solved these challenges. 


## Challenges

### 1. Baby Something

#### Brief

We are provided the following message: 

> How does the song go again? Baby *something* do do do do do do...

Alongside this, we are provided a .pcap file named *babysomething.pcap,* thus based on the lyrics heavily suggesting that *something* is "shark," we most likely have to analyze the .pcap using Wireshark.

### Info Gathering

Upon opening the .pcap file, we're provided with the following network traffic: 
![network traffic](\assets\images\August2025MetaCTF\NetworkTraffic.png)




### 2. Super Quick Logic Invitational

## Conclusion

Now that I'm in my third year at Sheridan, I've gotten quite busy with school, so no promises as to how often I will update my page!