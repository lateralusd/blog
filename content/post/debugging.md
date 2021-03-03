+++
title = "Debugging iOS apps on jailbroken iPhone"
description = "LLDB + debugserver to debug iOS apps"
tags = [
	"lldb",
	"iPhone",
	"debugging",
	"debugserver",
]
date =  2020-07-03T13:44:28+02:00
author = "XdaemonX"
+++

The first time I have tried to debug iOS apps on my jailbroken iPhone I hit the wall. There were many issues I had to solve so in this short post I'm gonna try to help you with this. Since you came to this post, I believe you already know what is lldb so I won't talk about it and let's get straight to the point.

__Prerequisites:__
1. Jailbroken iPhone (mine is iPhone 11, 13.4.1)
2. OpenSSH installed and running on iPhone since we will be using `scp` to copy files to and from device.

## Getting debugserver on device
The first thing you need to do is get `debugserver` on the device. There are generally two ways you can do this:
* Using hdiutil to attach certain image for your iOS
* Creating simple blank iOS app in Xcode and deploying it to your device.

You can check whether you successfully did any of these methods by checking if there is file `/Developer/usr/bin/debugserver`. If you see it, procede to the next section. Otherwise, repeat steps to get your debugserver on device.

## Signing debugserver
The next thing you want to do is transfer `debugserver` to your Mac. I am using `scp` for transfering, you can use any tool you are comfortable with. I am connected to device over usb and _iproxy_ is up and running on port 2222 (`$ iproxy 2222 22`).

`$ scp -P 2222 root@localhost:/Developer/usr/bin/debugserver /tmp`

For signing debugserver I am using `ldid` which came installed with Theos. Create file __ent.xml__ with following content:

```<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>com.apple.springboard.debugapplications</key>
        <true/>
        <key>get-task-allow</key>
        <true/>
        <key>task_for_pid-allow</key>
        <true/>
        <key>run-unsigned-code</key>
        <true/>
</dict>
</plist>
```

Copy debugserver and ent.xml in the same folder and substitute `/opt/theos` for your install location of Theos.
After creating file, issue `$ /opt/theos/bin/ldid -Sent.xml debugserver`.

Create file ent.plist with content:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/ PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>com.apple.springboard.debugapplications</key>
	<true/>
	<key>run-unsigned-code</key>
	<true/>
	<key>get-task-allow</key>
	<true/>
	<key>task_for_pid-allow</key>
	<true/>
</dict>
</plist>
```

Then run `$ codesign -s - --entitlements ent.plist -f debugserver`. Copy debugserver back to the device, give it executive permission.

```
$ scp -P 2222 debugserver root@localhost:/usr/bin/debugserver
$ ssh root@localhost -p 2222
$ chmod u+x /usr/bin/debugserver
```

## Running debugserver and connecting with lldb

Spawn another iproxy instance which will be used for lldb connecting like so `$ iproxy 6666 6666` on your mac. Now it is time to run our debugserver, we will be attaching to SpringBoard. Open terminal app on your iPhone or connect over ssh and run `# /usr/bin/debugserver 127.0.0.1:6666 -a SpringBoard`. 

You should see output similar to the one below. Also, your iphone will seems like it is not responding, that is because debugserver is attached to it and waiting for lldb to connect it in order to continue the process.

```
# /usr/bin/debugserver 127.0.0.1:6666 -a SpringBoard
debugserver-@(#)PROGRAM:LLDB  PROJECT:lldb-900.3.104
 for arm64.
Attaching to process SpringBoard...
Listening to port 6666 for a connection from localhost...
```

At this point, you should have 2 or 3 tabs open:
1. `iproxy 2222 22`
2. `iproxy 6666 6666` - for debugserver
3. `ssh connection to iphone` - this applies only if you are connecting over ssh to iphone instead of terminal app

Now create another terminal tab and type `lldb` and then type `process connect connect://127.0.0.1:6666`. Output will look something like this:

```
(lldb) process connect connect://127.0.0.1:6666
Process 6984 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = signal SIGSTOP
    frame #0: 0x000000019c1ac784 libsystem_kernel.dylib`mach_msg_trap + 8
libsystem_kernel.dylib`mach_msg_trap:
->  0x19c1ac784 <+8>: ret

libsystem_kernel.dylib`mach_msg_overwrite_trap:
    0x19c1ac788 <+0>: mov    x16, #-0x20
    0x19c1ac78c <+4>: svc    #0x80
    0x19c1ac790 <+8>: ret
Target 0: (SpringBoard) stopped.
(lldb)
```

Type __c__ at lldb prompt to continue process and now your iphone will become responsive again. If you come this far, that means now you know how to debug ios applications on jailbroken iphone using lldb and debugserver. If you have any question, send me an email. 
