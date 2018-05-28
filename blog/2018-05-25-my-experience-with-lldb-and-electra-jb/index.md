---
title: My experience with LLDB on Electra Jailbreak 1.0.4 (second edition)
date: 2018-05-25
draft: false
toc: true
math: false
type: post
---

> The first edition of this article was written on March 18, 2018. This is the second edition, with some important updates.

I tried to google a short and clear instruction how to debug apps from AppStore on iOS devices jailbroken with [Electra](https://coolstar.org/electra/). I did not found anything useful, so I wrote this guide. It works for me, but I'm not sure it works for your. I tested it on 

* iPhone 7 running iOS 11.1.2 
* iPhone 5s running iOS 11.0.1

Both devices were jailbroken with Electra jailbreak 1.0.4. 

# Deploying `debugserver` from Xcode to your device

Firs of all, ssh your iOS device and look if `/Developer/usr/bin/debugserver` exists. If it doesn't, 

1. Run Xcode on your Mac
2. Open any ObjC project for iOS (or create a new one from scratch)
3. Keep Xcode opened. Plug your iOS device to USB.

As result, you should see in Xcode:

![](preparing_debug.png)

Wait until "Preparing debugging support for iPhone" finished. Then check `/Developer/usr/bin/debugserver` on your device. The `debugserver` binary should exist now.

# Debugging via USB

For me, it works only if I do debugging via USB. If `iproxy` is not installed on your Mac, install it, e.g. with `brew`:

```
$ brew install usbmuxd
```

Then run in your Mac console:

```
$ iproxy 6666 6666 &
$ iproxy 2222 22 &
```

Finally, attach you iPhone to USB. That's it, we are ready to start.

# Attaching LLDB to already running process

On your Mac console, connect the iPhone:

```
$ ssh -p 2222 root@localhost
```

In the iPhone's console, run

```
# ps ax
```

Find pid of the process you want to attach. Then run

```
# /electra/jailbreakd_client <the pid> 1
# /Developer/usr/bin/debugserver localhost:6666 -a <the pid>
```

If you see something like

```
debugserver-@(#)PROGRAM:debugserver  PROJECT:debugserver-360.0.26.14
 for arm64.
Attaching to process 1418...
Listening to port 6666 for a connection from localhost...
```

everything going well. Now, open another console on your Mac, and run

```
$ lldb
```

In LLDB console, run

```
(lldb) platform select remote-ios
(lldb) process connect connect://localhost:6666
```

That's it! :)

# Running an app under LLDB

On your Mac console, connect the iPhone:

```
$ ssh -p 2222 root@localhost
```

In the iPhone's console, run

```
# /Developer/usr/bin/debugserver localhost:6666 -x backboard /var/containers/Bundle/Application/<path to the app binary>
```

If you see something like

```
debugserver-@(#)PROGRAM:debugserver  PROJECT:debugserver-360.0.26.14
 for arm64.
Listening to port 6666 for a connection from localhost...
```

everything going well. Now, open another console on your Mac, and run

```
$ lldb
```

In LLDB console, run

```
(lldb) platform select remote-ios
(lldb) process connect connect://localhost:6666
```

If you face an error, 

1. Run the app without debugger
2. Attach debugger to the app as described in the previous section
3. Close the app with `(lldb) kill`
4. Try to run the app under debugger again

Maybe it helps.

Happy debugging!
