+++
title = "Code injection on nonjailbroken iPhone"
description = "Using custom dylib to modify applications on nonjailbroken iPhone"
tags = [
	"ios",
	"iphone",
	"code injection",
	"dylib"
]
date = 2020-12-09T19:31:28+01:00
author = "XdaemonX"
+++

## Introduction
A few days ago I was writing about injecting FridaGadget.dylib inside application on nonjailbroken device so I was thinking why not to do the same thing, just with custom code (code injection).

The idea was to change some functionality of original application using dynamic library (dylib). Links for application are on my github and links are below.

* [https://github.com/XdaemonX/WillGetHacked](https://github.com/XdaemonX/WillGetHacked)
* [https://github.com/XdaemonX/Haxor.dylib](https://github.com/XdaemonX/Haxor.dylib)

Like in my previous [post](https://xdaemonx.github.io/frida_patching/), the same prerequsities apply.

## Details about application
Three important files are: 
* TestClass.h
* TestClass.m
* ViewController.m

Implementation of ViewController inside ViewController.m file can be seen below.
```
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"ECHO View loaded");
    TestClass *test = [[TestClass alloc] init];
    [test whoami];
}

@end
```

Inside we have a method viewDidLoad which logs "ECHO View loaded" and after that we create an object of class TestClass which is defined in TestClass.h file. Then we call method *whoami* on it.

TestClass.h file:

```
@interface TestClass : NSObject
-(void)whoami;
@end
```

TestClass.m file:

```
@implementation TestClass
- (void)whoami {
    NSLog(@"ECHO I am from the app");
}
@end
```

After we run the app, in logs we can see something like this.
```
$ idevicesyslog -m ECHO
[connected]
Dec  9 22:36:20 WillGetHacked[4272] <Notice>: ECHO View loaded
Dec  9 22:36:20 WillGetHacked[4272] <Notice>: ECHO I am from the app
```

## Details about dynamic library
For our dynamic library, we have simple interface with one instance method in it. That is the method that will replace *whoami* method from interface *TestClass*.

Haxor.h file:

```
@interface Haxor : NSObject
- (void)replace;
@end
```

Implementation of this interface is where the real fun is. Let's take a look at the code.
```
#import "Haxor.h"
#include <objc/runtime.h>

@implementation Haxor
+ (void)load{
    NSLog(@"ECHO Loading Dylib");
    
    Class thisClass = [self class];
    Class toReplaceClass = NSClassFromString(@"TestClass");
    
    SEL selReplace = @selector(replace);
    SEL selOriginal = @selector(whoami);
    
    Method replaceMethod = class_getInstanceMethod(thisClass, selReplace);
    Method originalMethod = class_getInstanceMethod(toReplaceClass, selOriginal);
    
    IMP impReplace = method_getImplementation(replaceMethod);
    
    method_setImplementation(originalMethod, impReplace);
}

- (void)replace{
    NSLog(@"ECHO I am from the dylib");
}
@end

```

## Explanation
We will use what is called method swizzling. It basically uses power of objective c runtime to modify application.

```
Class thisClass = [self class];
Class toReplaceClass = NSClassFromString(@"TestClass");
```
Here we are taking Class objects from classes which have methods we would like to replace. In this case, we have method -[thisDylib replace] and method -[TestClass whoami]. Since we are in Haxor we can use `[self class]` to get it's class method. To obtain Class for the one we would like to replace, we call function `NSClassFromString` function which accepts NSString as class name and returns Class object for that class.

```
SEL selReplace = @selector(replace);
SEL selOriginal = @selector(whoami);
```

We are creating selector variables of methods we are gonna substitute.

```
Method replaceMethod = class_getInstanceMethod(thisClass, selReplace);
Method originalMethod = class_getInstanceMethod(toReplaceClass, selOriginal);
```

`class_getInstanceMethod` returns to us Method object of instance methods which we want. `replaceMethod` will basically have -[Haxor replace] as a Method object, while `originalMethod` will have -[TestClass whoami].

```
IMP impReplace = method_getImplementation(replaceMethod);
```

`impReplace` will have implementation of replaceMethod, or -[Haxor replace].

```
method_setImplementation(originalMethod, impReplace);
```

This is the last step, we are setting implementation of -[TestClass whoami] to be the same as -[Haxor replace].

## Creating ipa and installing it
The process is similar to the one we did last time.

Since Haxor is a static library (.a extension), we will first convert it to dynamic library (dylib) so we can inject it the same way we did with out FridaGadget.dylib.

My library is called libHaxor.a, so to convert it to dylib we use:

`$ xcrun --sdk iphoneos clang -arch arm64 -shared -all_load -o Haxor.dylib libHaxor.a`.

Make sure to replace `-arch` parameter with the architecture of your device.

Let's create Payload directory and copy our newly created dylib and app inside of it. 

**NOTE: You need to have valid provisioning profile in order to sign the app**

```
$ mkdir Payload/
$ cp -r WillGetHacked.app Payload/
$ cp Haxor.dylib Payload/WillGetHacked.app/
$ tree
.
├── Haxor.dylib
├── Payload
│   └── WillGetHacked.app
│       ├── Base.lproj
│       │   ├── LaunchScreen.storyboardc
│       │   │   ├── 01J-lp-oVM-view-Ze5-6b-2t3.nib
│       │   │   ├── Info.plist
│       │   │   └── UIViewController-01J-lp-oVM.nib
│       │   └── Main.storyboardc
│       │       ├── BYZ-38-t0r-view-8bC-Xf-vdC.nib
│       │       ├── Info.plist
│       │       └── UIViewController-BYZ-38-t0r.nib
│       ├── Haxor.dylib
│       ├── Info.plist
│       ├── PkgInfo
│       ├── WillGetHacked
│       ├── _CodeSignature
│       │   └── CodeResources
│       └── embedded.mobileprovision
├── embedded.mobileprovision
└── libHaxor.a
```

Now we need to insert our Haxor.dylib inside binary and add rpath so binary knows where to search for dylibs.

```
$ insert_dylib --strip-codesig --inplace @executable_path/Haxor.dylib Payload/WillGetHacked.app/WillGetHacked
Added LC_LOAD_DYLIB to Payload/WillGetHacked.app/WillGetHacked
$ install_name_tool -add_rpath @executable_path/. Payload/WillGetHacked.app/WillGetHacked
```

Copy your provisioning profile in the same directory where Payload/ directory is and run:

```
$ zip -qry hacked.ipa Payload/
$ applesign -m ./embedded.mobileprovision hacked.ipa
[ ... REDACTED ... ]
Target is now signed: hacked-resigned.ipa
Cleaning up /Users/daemon1/injection/hacked.ipa.f3b6be03-336c-44d7-8537-d96cf8281d2b
```

Installing on the device using `ideviceinstaller`

` $ ideviceinstaller -i hacked-resigned.ipa`

If everything is done correctly, application will get installed. Now if we check syslog, we will see that our dylib successfuly substituted method.

```
$ idevicesyslog -m ECHO
Dec  9 22:49:17 WillGetHacked(Haxor.dylib)[4302] <Notice>: ECHO Loading Dylib
Dec  9 22:49:17 WillGetHacked[4302] <Notice>: ECHO View loaded
Dec  9 22:49:17 WillGetHacked(Haxor.dylib)[4302] <Notice>: ECHO I am from the dylib
```

We can expand this even more, we can also add our custom methods to classes or perhaps calling original method after substituted one gets called.
