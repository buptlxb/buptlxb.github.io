---
layout: post
title: "Ubuntu 12.04 Ethernet Controller Problem"
description: "Ethernet controller can not detect the cable"
category: "Linux"
tags: ["Linux", "Network", "Ubuntu"]
tagline: "on Lenovo-Zhaoyang-E49 PC"
---

## Problem

Ethernet controller can not detect the cable.

## Solution

``` bash

wget http://r8168.googlecode.com/files/r8168-8.037.00.tar.bz2
tar xjf r8168-8.037.00.tar.bz2
cd r8168-8.037.00
./autorun.sh

```

{% include JB/setup %}
