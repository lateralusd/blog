+++
title = "SSL pinning is not that hard... sometimes"
description = "Bypassing SSL pinning"
tags = [
	"ssl",
	"ios",
	"iPhone",
]
date = 2021-02-22T17:22:16+01:00
author = "XdaemonX"
+++

# Introduction
Recently I was pentesting an iOS application which of course had SSL pinning enables. For those who are not familiar what is SSL pinning, let's take description from [https://thesslstore.com](https://thesslstore.com):

> Pinning is an optional mechanism that can be used to improve the security of a service or site that relies on SSL Certificates. Pinning allows you to specify a cryptographic identity that should be accepted by users visiting your site.

Basically, SSL pinning is checking to see whether the application is communicating with the right server. SSL pinning is an obstacle that has to be removed if you want to intercept the request which application is making.

Some of the common methods to bypass SSL pinning are using [objection](https://github.com/sensepost/objection) or using [SSL Kill Switch 2](https://github.com/nabla-c0d3/ssl-kill-switch2).

# Bypassing
This was all done on my iPhone running iOS version 14.4 nonjailbroken so immediately SSL Kill Switch 2 is not an option because it is working only on jailbroken devices and we are left with objection.

After patching the application with FridaGadget inside(you can read how to do it [here](/frida_patching/)) we can use objection.

```
$ objection -g Gadget explore

```
