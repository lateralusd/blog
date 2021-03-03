+++
title = "FridaGadget.dylib on nonjailbroken iPhone"
description = "Finally got it"
tags = [
	"ios",
	"iphone",
	"frida",
]
date = 2020-12-05T22:31:11+01:00
author = "XdaemonX"
+++

## Introduction
After trying for 7.30 hours to insert FridaGadget into the application, I have finally did it. It took reading bunch of tutorials and trials until it all clicked. 

Anyway, FridaGadget allows us instrumentation (lets say hacking) of an application on non jailbroken iPhone. I was using my iPhone which is running iOS 14.0.1.

## Prerequisities
* [Frida Gadget](https://github.com/frida/frida/releases/tag/14.1.2)
* [insert_dylib](https://github.com/Tyilo/insert_dylib)
* [applesign](https://github.com/nowsecure/node-applesign)
* [ios-deploy](https://github.com/ios-control/ios-deploy)
* [Frida](https://github.com/frida/frida)

## Preparation

The first thing you need it to get provisioning profile, or let's call it your certificate which will be used to sign an application. To do this, create an empty project, give it a name, a **remember** its bundle identifier. After that, build an app and in the XCode under the Project Navigator, you should see directory *Products* and in it .app file.

Right click on the .app file and click on *Show in Finder*. After the new Finder window opens, right click on an app and choose *Show Package Contents*. Copy file **embedded.mobileprovision** file on some location on your system, lets say */tmp/* directory.  
`$ cp embedded.mobileprovision /tmp`

## Patching the application
First download the gadget from the site given in the prerequisites section. After you have done that, copy .ipa file, embedded.mobileprovision and frida gadget to the same directory, like in an example below.

```
$ tree /tmp/patchingIPA/
.
├── FridaGadget.dylib
├── embedded.mobileprovision
└── testApp.ipa

0 directories, 3 files
```

Unzip the .ipa file and copy FridaGadget.dylib inside it.

```
Archive:  testApp.ipa
   creating: Payload/
   creating: Payload/testApp.app/
   creating: Payload/testApp.app/_CodeSignature/
  inflating: Payload/testApp.app/_CodeSignature/CodeResources
   creating: Payload/testApp.app/Base.lproj/
   creating: Payload/testApp.app/Base.lproj/Main.storyboardc/
  inflating: Payload/testApp.app/Base.lproj/Main.storyboardc/UIViewController-BYZ-38-t0r.nib
  inflating: Payload/testApp.app/Base.lproj/Main.storyboardc/BYZ-38-t0r-view-8bC-Xf-vdC.nib
  inflating: Payload/testApp.app/Base.lproj/Main.storyboardc/Info.plist
   creating: Payload/testApp.app/Base.lproj/LaunchScreen.storyboardc/
  inflating: Payload/testApp.app/Base.lproj/LaunchScreen.storyboardc/01J-lp-oVM-view-Ze5-6b-2t3.nib
  inflating: Payload/testApp.app/Base.lproj/LaunchScreen.storyboardc/UIViewController-01J-lp-oVM.nib
  inflating: Payload/testApp.app/Base.lproj/LaunchScreen.storyboardc/Info.plist
  inflating: Payload/testApp.app/embedded.mobileprovision
  inflating: Payload/testApp.app/testApp
  inflating: Payload/testApp.app/Info.plist
 extracting: Payload/testApp.app/PkgInfo
$ cp FridaGadget.dylib Payload/testApp.app/
```

Now we need to tell our binary to load this FridaGadget.dylib, we do that using `insert_dylib`.

```
$ insert_dylib --strip-codesig --inplace @executable_path/FridaGadget.dylib Payload/testApp.app/testApp
Added LC_LOAD_DYLIB to Payload/testApp.app/testApp
```

There is one more thing left before packaging our app and deploying it, and that is telling our binary where to search for dylibs. In rescue comes `install_name_tool`.

```
$ install_name_tool -add_rpath @executable_path/. Payload/testApp.app/testApp
```

## Signing application

For signing, we will use applesign. The first thing we need to do is package our Payload/ directory into new app.

`$ zip -qry patched.ipa Payload/`

Now we run applesign on it using our embedded.mobileprovision file and passing the bundle identifier which we setup during creation of our test app. In my case it was com.delorean.test.

```
$ applesign --mobileprovision ./embedded.mobileprovision --bundleid com.delorean.test patched.ipa
File: /private/tmp/testApp/test/patchingIPA/patched.ipa
[ ... REDACTED ... ]
Target is now signed: patched-resigned.ipa
Cleaning up /private/tmp/testApp/test/patchingIPA/patched.ipa.afdd2d36-e576-42b4-beaa-b1a3f9eec14e
```

As applesign says, our signed app is now inside *patched-resigned.ipa*. We are gonna create a new directory, copy our new resigned app in it, unzip it and deploy it to our device.

```
$ mkdir test
$ cp patched-resigned.ipa test && cd test
$ unzip patched-resigned.ipa
Archive:  patched-resigned.ipa
   creating: Payload/
   creating: Payload/testApp.app/
   creating: Payload/testApp.app/_CodeSignature/
	[ ... REDACTED ... ]
$ ideviceinstaller -i patched-resigned.ipa
WARNING: could not locate iTunesMetadata.plist in archive!
WARNING: could not locate Payload/testApp.app/SC_Info/testApp.sinf in archive!
Copying 'patched-resigned.ipa' to device... DONE.
Installing 'com.delorean.test'
Install: CreatingStagingDirectory (5%)
Install: ExtractingPackage (15%)
Install: InspectingPackage (20%)
Install: TakingInstallLock (20%)
Install: PreflightingApplication (30%)
Install: InstallingEmbeddedProfile (30%)
Install: VerifyingApplication (40%)
Install: CreatingContainer (50%)
Install: InstallingApplication (60%)
Install: PostflightingApplication (70%)
Install: SandboxingApplication (80%)
Install: GeneratingApplicationMap (90%)
Install: Complete
```

When you click on app, it will say that it is Untrusted Developer, go to the Settings -> General -> Profile & Device Management, choose developer and trust it.

After you have done that, run ios-deploy.

```
$ ios-deploy --bundle Payload/testApp.app/ -W -d
[ ... REDACTED ...]
(lldb)     connect
(lldb)     run
success
2020-12-05 23:13:03.221242+0100 testApp[668:64572] Frida: Listening on 127.0.0.1 TCP port 27042
```

In other window, run `$ frida-ps -U | grep -i gadget`. If everything is done correctly, you should see it in the listing and you can connect to it and hack the hell out of it.

```
$ frida-ps -U | grep -i gadget
668  Gadget
$ frida -U Gadget
     ____
    / _  |   Frida 14.0.8 - A world-class dynamic instrumentation toolkit
   | (_| |
    > _  |   Commands:
   /_/ |_|       help      -> Displays the help system
   . . . .       object?   -> Display information about 'object'
   . . . .       exit/quit -> Exit
   . . . .
   . . . .   More info at https://www.frida.re/docs/home/
```
