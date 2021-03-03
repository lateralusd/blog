+++
title =  "Connecting the dots between Theos and Cycript/Cyrun"
description = "What is the releationship between the two"
tags = [
	"theos",
	"cycript",
	"cyrun",
]
date = 2020-07-03T17:28:54+02:00
author = "XdaemonX"
+++

We have all been playing with cycript and changing those labels or perhaps we hid some views but we did not see the bigger picture of all of it, and the biggest question of them all is how it is related to __class-dump__ and __theos__. To make things clear, im gonna take local news app.

## Theos 

Lets take definition from [iphonedevwiki](http://iphonedevwiki.net/index.php/Theos) which says:
> Theos is a cross-platform suite of development tools for managing, developing, and deploying iOS software without the use of Xcode.

What this means is that you can override certain app funcionality, extend it or remove it completely. Result of theos is called *tweak*. Let's say you have a functionality in app which shows advertisements, you could make a theos tweak (override method) not to show those advertisements.

## Class-dump
Class-dump tool or alternative version class-dump-z which is faster is basically what its name says. It allows you to dump all classes as well as their methods inside the specific app binary. I like to install it on my Mac and run everything from there because it is easier for me. The first thing I do is dump application using tools like __frida-ios-dump__ or __Clutch__, extract the *.ipa* file and run the class-dump against the binary.

Let's say app's binary is called _NotMe_, good combination of flags would be:  
`$ class-dump -S -s -H NotMe -o /tmp/NotMeHeaders`

Run `class-dump --help` to see what each options means.

## Cycript or cyrun, what the hell
The name of the original tool is called __cycript__ which was made by saurik himself. On certain versions of jailbreak cycript does not work, and there is alternative called _cyrun_.

Cycript enables you to attach to the application and inspect it in runtime. You can view all those views, labels, textfields etc. You can manipulate those elements however you want according to ObjC rules. Cycript is also useful to find which view controller is responsible for certain element, or which methods gets called on let's say button click.

## Connecting everything
