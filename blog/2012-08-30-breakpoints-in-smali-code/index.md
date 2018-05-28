---
title: Breakpoints in Smali code
date: 2012-08-30
draft: false
toc: false
math: false
type: post
---

In [my previous posts](../2012-08-27-debugging-smali-code-with-apk-tool-and-netbeans/) I wrote a step-by-step instruction how to debug Smali code with Apktool and NetBeans. However, there were no details about breakpoints, just a short note in steps 12 and 13

> 12.&nbsp;Set a breakpoint. You must select line with some instruction, you can't set breakpoint on lines starting with `.`, `:` or `#`.
>
> 13.&nbsp;Trigger some action in application. If you run at the breakpoint, then the thread should stop and you will be able to debug it step by step, watch variables, etc.

In this post I'm going to share more information about breakpoints in Smali code.

Using Apktool and NetBeans for Smali debugging, we usually face a problem. We connect the debugger to an already running application. Some Smali code has already been executed _before_ we connected the debugger, and will never be executed again. For example, the constructor and `onCreate()` method of the main activity class. We can't debug such code, so we may miss something important at the beginning of the application.

To solve the problems, we can use several ways. The easiest one is `waitForDebugger()`. Here is the step-by-step instruction: 

1. Follow [the instruction](../2012-08-27-debugging-smali-code-with-apk-tool-and-netbeans/debugging-smali-code-with-apk-tool-and-netbeans.md), do steps 1 and 2.

2. In `out/AndroidManifest.xml`, find the activity with the following filter

		<intent -filter="-filter">
			<action android:name="android.intent.action.MAIN">
			<category android:name="android.intent.category.LAUNCHER">
		</intent>

3. Find the activity class, e.g. `Lmy/activity/class/MyActivity;`. Then find the `onCreate(...)` method (the activity class is a descendant of `android.app.Activity`, it overrides the `onCreate(...)` method). The `onCreate(...)` method is executed after the constructor of the activity class. In most cases, an application logic starts in `onCreate(...)`.

4. Insert

		invoke-static {}, Landroid/os/Debug;->waitForDebugger()V

	after

		invoke-super {p0, p1}, Landroid/app/Activity;->onCreate(Landroid/os/Bundle;)V

	in the `onCreate(...)` method.

5. Again, follow [the instruction](../2012-08-27-debugging-smali-code-with-apk-tool-and-netbeans/), do steps from 3 to 11. You will see just a black screen after you start the application on step 9. __Don't worry, it's normal. If Android proposes you to close the application because that the application does not respond, reject the proposition.__ The application is frozen at the very beginning of execution, it is waiting for the debugger.

6. Set a breakpoint on the first instruction after

		invoke-static {}, Landroid/os/Debug;->waitForDebugger()V

	in the `onCreate(...)` method and continue execution. Your breakpoint will be hit and you will be able to debug the application step by step from the very beginning, watch variables, etc.

From my experience, `waitForDebugger()` does not work in some cases. Well, it's not a big problem because we can insert an infinite loop instead of `waitForDebugger()` in the `onCreate(...)` method. We should just follow the step-by-step instruction above, but use

```
:debug
sget v0, Lmy/activity/class/MyActivity;->debugFlag:I
if-nez v0, :debug    
```

code instead of 

```
invoke-static {}, Landroid/os/Debug;->waitForDebugger()V
```

Do not forget to add the field 

```
.field static debugFlag:I = 0x01
```

to the class. As soon as the `onCreate(...)` method invoked, it goes to an infinite loop while `debugFlag` is not 0. So, we have a lot of time to connect the debugger to the running application, then use the debugger to set a breakpoint on the first Smali instruction after the loop, and change `debugFlag` to `0` to escape from the infinite loop. If everything is well-done, our breakpoint is hit and we can start debugging from the very beginning of the application. 

That's it :) Any questions are wellcome.