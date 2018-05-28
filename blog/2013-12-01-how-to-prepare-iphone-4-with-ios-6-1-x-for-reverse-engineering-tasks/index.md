---
title: How to prepare iPhone 4 with iOS 6.1.x for reverse engineering tasks
date: 2013-12-01
draft: false
toc: false
math: false
type: post
---

Here is an extremely detailed step-by-step instruction for a jailbroken iPhone 4 with iOS 6.1.x and Cydia installed. It will be useful for beginners, not for advanced reverse engineers.

# Step 1. OpenSSH

First of all, install OpenSSH. On the iPhone, run Cydia and tap "OpenSSH Access How-To":

![](001.png)

Tap the "OpenSSH" link:

![](002.png)

Tap "Install"

![](003.png)

and "Confirm":

![](004.png)

Wait until installation is completed and tap "Return to Cydia":

![](005.png)

Now OpenSSH is installed. On the iPhone, open "Settings" → "Wi-Fi" and connect the iPhone to a Wi-Fi network (if it is not connected yet). Then tap the blue arrow icon next the Wi-Fi network name ![](006.png) and note the IP address of the iPhone: 

![](007.png)

On a PC, connected to the same Wi-Fi network, run your favorite SSH client ([Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/) or whatever you use) and connect to the IP address you noted above. Use `root` as the login and `alpine` as the password. Enter `passwd` commands in the SSH console to change root and mobile passwords: 

```
iPhone:~ root# passwd
Changing password for root.
New password: <type your new password here>
Retype new password: <retype your new password here>
	
iPhone:~ root# passwd mobile
Changing password for mobile.
New password: <type your new password here>
Retype new password: <retype your new password here>
```

# Step 2. GDB

It's time to install GDB. On the iPhone, run Cydia and tap "Manage" and "Sources":

![](008.png)

Tap "Edit"

![](009.png)

and "Add":

![](010.png)

Enter <http://cydia.myrepospace.com/ohmza/> and tap "Add Source" 

![](011.png)

Wait until installation is completed and tap "Return to Cydia": 

![](012.png) 

Tap "Sections", choose "ohmza -myRepoSpace": 

![](013.png) 

Tap "GNU Debugger (iOS 5&6)": 

![](014.png) 

Then tap "Install" 

![](015.png)

and "Confirm"

![](016.png)

Wait until installation is completed and tap "Return to Cydia": 

![](017.png)

That's it! 

# Step 3. *nix commands

To make well-known *nix commands (e.g. `ps`) working on the iPhone, install the `adv-cmds` package from Cydia. On the iPhone, run Cydia and tap "Sections" and "Administration":

![](018.png)

Then tap "adv-cmds"

![](019.png)

and "Install" 

![](020.png) 

and, finally, "Confirm" 

![](021.png) 

Wait until installation is completed and tap "Return to Cydia": 

![](022.png) 

That's all. All links in the text were alive when posting.