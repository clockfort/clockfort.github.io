---
layout: post
title: "SDK Troubles"
date: 2013-08-11 22:50
comments: true
categories: [linux,porting,hacking]
---

Cavium apparently requires you to buy an expensive hardware development kit in order to get access to their SDK, which provides a simulator / base linux image / compiler for their platform. Bummer. After a while of digging though, a helpful wiki page from the FreeBSD MIPS team indicated that one may also petition to join a fairly closed-door community called [cnUsers](http://cnusers.org/), where approved members are allowed access to a pretty crippled occasionally released copy of the Cavium SDK. However, it does not offer advanced crypto/compression library support, and more importantly does not include the simulator/base linux image. 😢

Regardless, I have applied, and eagerly await evaluating what help, if any, the open source SDK will be in any porting efforts.
