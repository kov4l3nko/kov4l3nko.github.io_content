---
title: Installing LLDB
date: 2014-02-14
draft: false
toc: false
math: false
type: post
---

---

__Warning!__ At the moment, the guide is obsolete. It makes sense to read [the updated and extended version of this guide](../2016-04-27-debugging-ios-binaries-with-lldb/).

---

Today we will talk about installation of LLDB (on Mac OS) and debugserver (on iOS 7.0-7.0.4).

# Installation

You need LLDB installed on your Mac and the debugserver installed on your iOS device. To install LLDB on your Mac, just install the latest version of [XCode](https://developer.apple.com/xcode/). To install the debugserver on iOS device, mount `DeveloperDiskImage.dmg` and copy `debugserver` to a local folder on your Mac: 

```
$ hdiutil attach /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/7.0.3\ \(11B508\)/DeveloperDiskImage.dmg
$ cp /Volumes/DeveloperDiskImage/usr/bin/debugserver .
```

Then create `entitlements.plist`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/ PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>com.apple.springboard.debugapplications</key> <true/>
	<key>run-unsigned-code</key>
	<true/>
	<key>get-task-allow</key> <true/> <key>task_for_pid-allow</key> <true/>
</dict> 
</plist>
```

and use it to sign `debugserver`

```
$ codesign -s - --entitlements entitlements.plist -f debugserver
```

Finally, copy the signed `debugserver` to `/usr/bin/` on your iOS device (here and further we assume that the IP address of the device is `192.168.1.110`): 

```
$ scp ./debugserver root@192.168.1.110:/usr/bin
```

It's done! 

# Check if it works

To check if LLDB+debugserver work correctly, SSH your iOS device and attach debugserver to a running process (in the example below, the process is Instagram, but you can attach debugserver to any process): 

```
# debugserver *:6666 -a Instagram
```

As result, in your SSH console you should see something similar to 

```
debugserver-300.2 for armv7.
Attaching to process Instagram...
Spawning general listening thread.
Spawning kqueue listening thread.
Listening to port 6666 for a connection from *...
```

Now, start LLDB in your Mac console and connect debugserver: 

```
$ lldb
(lldb) platform select remote-ios
(lldb) process connect connect://192.168.1.110:6666
```

In your SSH console you'll see: 

```
Waiting for debugger instructions for process 0.
```

Wait, wait, wait… it can take 1-3 minutes to connect (why so long? I don't know). Finally, in your Mac console you should see something like 

```
Process 2962 stopped
* thread #1: tid = 0x1902c, 0x37db4a8c libsystem_kernel.dylib`mach_msg_trap + 20, queue = 'com.apple.main-thread, stop reason = signal SIGSTOP
    frame #0: 0x37db4a8c libsystem_kernel.dylib`mach_msg_trap + 20
libsystem_kernel.dylib`mach_msg_trap + 20:
-> 0x37db4a8c:  pop    {r4, r5, r6, r8}
   0x37db4a90:  bx     lr
   
libsystem_kernel.dylib`mach_msg_overwrite_trap:
   0x37db4a94:  mov    r12, sp
   0x37db4a98:  push   {r4, r5, r6, r8}
(lldb)
```

This means LLDB+debugserver work just fine and you can start debugging. If you need more information about attaching debugserver to already running process, check the next section. 

# It doesn't work, what to do?

Sometimes `debugserver` works incorrectly or does not work at all becuase of permissions. SSH your iOS device and check  `debugserver` permissions: 

```
# ls -l /usr/bin/debugserver
```

The correct permissions are 

```
-rwxr-xr-x 1 root wheel 1100912 Feb 13 15:30 /usr/bin/debugserver
```
