---
title: Debugging Smali code with apk-tool and NetBeans
date: 2012-08-27
draft: false
toc: false
math: false
type: post
---

Ho-ho-ho, it works now! Here is a (more or less detailed) instruction to debug a Smali code of a third-part Android application. To debug Smali code with apk-tool, you need

1. [Apk-tool 1.4.1](http://code.google.com/p/android-apktool/downloads/detail?name=apktool1.4.1.tar.bz2&can=1&q=) and [NetBeans 6.8](http://netbeans.org/downloads/6.8/index.html). **Use these versions, not the latest ones! Currently, the latest versions of apk-tool and NetBeans do not allow to debug Smali code.**

2. Java, JDK and other stuff installed in your system to make Apk-tool and  NetBeans working

The step-by-step instruction: 

1. Decode your .apk file to `out` directory, use `-d` option:`java -jar apktool.jar d -d my.app.apk out

2. Add `android:debuggable="true"` attribute to `<application>` section in `out/AndroidManifest.xml` file.

3. Build `out` directory to .apk file:

    ```
    $ java -jar apktool.jar b -d out my.app.to.debug.apk
    ```

4. Sing and install `my.app.to.debug.apk` to the Android device used for debugging

5. Delete `out/build` folderRun NetBeans, click "File" --> "New Project". Choose "Java" --> "Java Project with Existing Sources". Click "Next".

6. Specify `out` as "Project Folder". Click "Next".

7. Add `out/smali` folder to the "Source Package Folder" list. Click "Next" and then "Finish".

8. Start `my.app.to.debug.apk` on the device, run DDMS, find your application on a list and click it. Note port information in last column - it should be something like `86xx / 8700`.

9. In NetBeans, click "Debug" --> "Attach Debugger" --> select "JPDA" and set "Port" to `8700` (or whatever you saw in previous step). Rest of fields should be ok, click "OK".

10. Debugging session should start: you will see some info in a log and debugging buttons will show up in top panel.

11. Set a breakpoint. You must select line with some instruction, you can't set breakpoint on lines starting with `.`, `:` or `#`.

12. Trigger some action in application. If you run at breakpoint, then thread should stop and you will be able to debug step by step, watch variables, etc.

I copy-pasted steps 9-13 from [the original instruction](http://code.google.com/p/android-apktool/wiki/SmaliDebugging) (sorry, English is not my native language... and I'm too lazy to "produce unique content for the blog"™, so just copy-pasted description of last 5 steps :)).

Questions are welcome. Have a nice day :)

P.S.  If you have problems with breakpoints, [this](../2012-08-30-breakpoints-in-smali-code/) may help.