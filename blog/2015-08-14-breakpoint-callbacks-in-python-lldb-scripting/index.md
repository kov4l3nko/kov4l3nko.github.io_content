---
title: Breakpoint callbacks in LLDB Python scripts
date: 2015-08-14
draft: false
toc: false
math: false
type: post
---

Today we talk about breakpoint callbacks in LLDB Python scripts. Setting breakpoints is easy. For example, write the Python script:

```
# This is test.py 
	
import lldb
	
def test(debugger, command, result, internal_dict):
    """ Just a test command to set a breakpoint """
    target = debugger.GetSelectedTarget()
    breakpoint = target.BreakpointCreateByName("SSLWrite")
	
def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand('command script add -f test.test test')
```

The scripts implements a command `test`. The command sets a breakpoint on `SSLWrite`.

Name the file `test.py`, save it to a hard drive. Then ssh your iOS device and run `debugserver` on the device. Then run LLDB on the host and run the `process connect ...` command to connect the device. Finally, load the script, run `test`, and list breakpoints:

```
(lldb) command script import path/to/test.py
(lldb) test
(lldb) breakpoint list 
Current breakpoints:
2: name = 'SSLWrite', locations = 1, resolved = 1, hit count = 0
  2.1: where = Security`SSLWrite, address = 0x335fb4dc, resolved, hit count = 0
```

See? The breakpoint was set, it works :) Now lets make the debugger doing something after the breakpoint hits. For this purpose, we can use _a callback_.  Let's modify the script:

```
# This is test.py 

import lldb
	
def test(debugger, command, result, internal_dict):
    """ Just a test command to set a breakpoint """
    target = debugger.GetSelectedTarget()
    breakpoint = target.BreakpointCreateByName("SSLWrite")
    # add a callback to the breakpoint
    breakpoint.SetScriptCallbackFunction('test.breakpoint_callback')
	
# This is the callback function, it does nothing useful but says "Hit!"
def breakpoint_callback(frame, bp_loc, dict):
    print "Hit!"
	
def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand('command script add -f test.test test')
```

Here, the main point is `breakpoint.SetScriptCallbackFunction(...)`. It looks like the method was introduced in the commit [r205494](https://www.mail-archive.com/lldb-commits@cs.uiuc.edu/msg01637.html). However, at the moment, it is _not yet documented_ in [the official LLDB Python reference](http://lldb.llvm.org/python_reference/index.html). And even if you know the exact method's name, Google gives you no more that 20 pages with almost useless info. I hope they will add the description of this cool method to the official docu soon.

Ok, reload the script and test the `test` command:

```
(lldb) breakpoint delete 
About to delete all breakpoints, do you want to do that?: [Y/n] y
All breakpoints removed. (1 breakpoint)
(lldb) test 
(lldb) breakpoint list 
Current breakpoints:
3: name = 'SSLWrite', locations = 1, resolved = 1, hit count = 0
    Breakpoint commands:
      return test.breakpoint_callback(frame, bp_loc, internal_dict)
	  
  3.1: where = Security`SSLWrite, address = 0x335fb4dc, resolved, hit count = 0  
```

Continue the process. As soon as the process calls `SSLWrite`, we see:

```
(lldb) c
Process 395 resuming
(lldb) Hit!
Process 395 stopped
* thread #5: tid = 0x7dd5, 0x335fb4dc Security`SSLWrite, name = 'com.apple.NSURLConnectionLoader', stop reason = breakpoint 3.1
    frame #0: 0x335fb4dc Security`SSLWrite
Security`SSLWrite:
->  0x335fb4dc <+0>: push   {r4, r5, r6, r7, lr}
    0x335fb4de <+2>: add    r7, sp, #0xc
    0x335fb4e0 <+4>: push.w {r8, r10, r11}
    0x335fb4e4 <+8>: sub    sp, #0xc
```

So it works :) You can download the script [here](test.py).