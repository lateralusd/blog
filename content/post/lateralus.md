+++
title = "Modlishka & Lateralus"
description = "Strange words"
tags = [
	"phishing",
	"tools",
	"golang",
]
date = 2020-10-08T14:54:09+02:00
author = "XdaemonX"
+++

## Introduction
Let's first start by giving description of Modlishka and lateralus.

### Modlishka
[Modlishka](https://github.com/drk1wi/Modlishka) is basically a reverse proxy which can be used to bypass 2FA, collect credentials and generally is helpful for phishing campaigns.  
It has a lot of options, such as:  
* Injecting custom javascript code (can be useful to rewrite parts of page) 
* Substitutions of strings
* Credentials collection
* Domain mode hijacking

Here I will show only some parts of it, you should download it and give it a try.

### Lateralus
[Lateralus](https://github.com/XdaemonX/lateralus) is a tool I have built specifically for phishing campaigns. It allows you to create email template, select targets, generate urls and correlate data with control db file of Modlishka. It is written in the same language as Modlishka, Golang, and as such it is easy to understand how it works.

## Goal
The goal is to collect credentials from out target, as a target site we will choose example.com.

## Setup
Our domain will be at lateralus.uk 
