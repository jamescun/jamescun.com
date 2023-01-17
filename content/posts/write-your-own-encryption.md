---
title: Write your own encryption
date: 2023-01-17T18:00:00+01:00
draft: false
---

> **NEVER** write your own encryption!

or so is often quipped online, and it's good advice - but it's not complete. It should be never _productionize_ your own encryption!

One of the best ways to learn, at least for me, is to fuck around and find out. Recently I've written my own implementations of TLS and SSH, both protocols we use daily but rarely introspect. Would I ever trust my own implementation in a real environment? **Absolutely not**. Do I have a better understanding of them, can make reasoned tradeoffs in their configuration, and more readily debug their more asinine issues? **Absolutely**.
