---
title: Bypassing an anti-debug protection in Musical.ly 4.7.2 for iOS
date: 2016-03-07
draft: false
toc: true
math: false
type: post
---

Today we disable anti-debug protection in Musical.ly 4.7.2 for iOS.

# Segmentation fault: 11

I installed the [musical.ly app](https://itunes.apple.com/us/app/musical.ly-your-music-video/id835599320) from AppStore on my jailbroken iPhone 4 to take a quick look on the app under debugger. I tried to attach `debugserver` to a running process and got

```
iPhone:~ root# debugserver *:6666 -a Musical.ly                                                                                     
debugserver-310.2 for armv7.
Attaching to process Musical.ly...
Segmentation fault: 11
```

So the app used some anti-debugging techniques. Interesting...

# Decrypting the binary

It was easy. Most of iOS anti-debugging trick work *after* the binary started. So I just

1. Killed the app process

2. Started the app binary under `debugserver` and got

    ```
    iPhone:~ root# debugserver *:6666 -x backboard /var/mobile/Applications/BA708472-5A17-41FB-B233-1B5DA84460D6/Musical.ly.app/Musical.ly 
    debugserver-310.2 for armv7.
    Listening to port 6666 for a connection from *...
    ```

3. Then, I run LLDB, connected it to the debug server, and listed modules:

    ```
    (lldb) image list
    [  0] 651A31C3-9F71-311F-965F-8AC44DE02C88 0x2bea6000 /Users/administrator/Library/Developer/Xcode/iOS DeviceSupport/7.1.2 (11D257)/Symbols/usr/lib/dyld
    ```

    `Musical.ly` not loaded. No modules loaded at all. Ok.

4. I loaded modules:
	
	    (lldb) settings set target.process.stop-on-sharedlibrary-events 1
	    (lldb) c
	    Process 1105 resuming
	    (lldb) error: libarclite_iphoneos.a(arclite.o) failed to load objfile for /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/arc/libarclite_iphoneos.a
	    Process 1105 stopped
	    * thread #1: tid = 0x8777f, 0x2be6d0cc dyld`gdb_image_notifier, stop reason = shared-library-event
	        frame #0: 0x2be6d0cc dyld`gdb_image_notifier
	    dyld`gdb_image_notifier:
	    ->  0x2be6d0cc <+0>: bx     lr
		
	    dyld`dyldbootstrap::start:
	        0x2be6d0d0 <+0>: push   {r4, r5, r6, r7, lr}
	        0x2be6d0d2 <+2>: add    r7, sp, #0xc
	        0x2be6d0d4 <+4>: push.w {r8, r10, r11}
	    (lldb)
    
	Image list said that `Musical.ly` was loaded

	```
	 (lldb) image list Musical.ly
	 [  0] 8BBD1E1B-A68B-32D5-8FAC-11C30CE33079 0x0006e000 /var/mobile/Applications/BA708472-5A17-41FB-B233-1B5DA84460D6/Musical.ly.app/Musical.ly (0x000000000006e000)
	```

Finally, I just did the trick described [here](../2016-03-01-decrypting-apps-from-appstore/), and got a decrypted binary. Yes, it was easy.

# Meet ADVobfuscator

Hopper disassembler said the app used [ADVobfuscator](https://github.com/andrivet/ADVobfuscator). I found a lot of traces of the obfuscator in symbols, e.g.

```
imp___picsymbolstub4___ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINS0_5front4noneEEENS1_11HandledEnumERKT_b:
0x00fb4d5c         ldr        ip, = 0x315db8                                    ; 0x315db8,0xfb4d68, XREF=__ZL5cleanPv+128, __ZL5cleanPv+276, __Z7p_beginPFvvE+858, __Z7p_beginPFvvE+1396, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINS0_5front4noneEEENS1_11HandledEnumERKT_b+46, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINSE_6event4EEENS1_11HandledEnumERKT_b+46, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINSE_6event5EEENS1_11HandledEnumERKT_b+46, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINSE_6event2EEENS1_11HandledEnumERKT_b+46, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINSE_6event3EEENS1_11HandledEnumERKT_b+46, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalISD_EENS1_11HandledEnumERKT_b+46, __ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINS0_5front4noneEEENS1_11HandledEnumERKT_b$shim+4, …
0x00fb4d60         add        ip, pc, ip                                        ; imp___la_symbol_ptr___ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINS0_5front4noneEEENS1_11HandledEnumERKT_b
0x00fb4d64         ldr        pc, [ip]                                          ; 0x466119,imp___la_symbol_ptr___ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine17MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEES8_EENS_9parameter5void_ESG_SG_SG_E22process_event_internalINS0_5front4noneEEENS1_11HandledEnumERKT_b
                        ; endp
0x00fb4d68         dd         0x00315db8
```

The ADVobfuscator produced a sophisticated machine code, but it was not a problem. I didn't need to crack it all, I just needed to switch anti-debugging off.

# AmIBeingDebugged? He-he...

After a quick look inside ADVobfuscator sources, I [found](https://github.com/andrivet/ADVobfuscator/blob/master/ADVobfuscator/DetectDebugger.cpp) `AmIBeingDebugged()` function. It looked like

```
bool AmIBeingDebugged()
// Returns true if the current process is being debugged (either running under the debugger or has a debugger attached post facto).
{
    // Note: It is possible to obfuscate this with ADVobfuscator (like the calls to getpid and sysctl)
    
    kinfo_proc info;
    info.kp_proc.p_flag = 0;
    
    int mib[] = {CTL_KERN, KERN_PROC, KERN_PROC_PID, getpid()};
   
    size_t size = sizeof(info);
    sysctl(mib, sizeof(mib) / sizeof(*mib), &info, &size, NULL, 0);
    
    return (info.kp_proc.p_flag & P_TRACED) != 0;
}
```


Hopper said the `Musical.ly` binary also had it

```
               __Z16AmIBeingDebuggedv:        // AmIBeingDebugged()
0x0044c434         push       {r4, r5, r7, lr}                                  ; XREF=__ZN5boost3msm4back13state_machineIN8andrivet13ADVobfuscator8Machine27MachineINS4_5eventINS4_4VoidENS4_17ObfuscatedAddressIPFvvEEEJEEE14DetectDebuggerS8_EENS_9parameter5void_ESH_SH_SH_E6a_row_INS0_5front3RowINSF_6State2ENSF_6event1ENSF_6State3ENSF_13CallPredicateENSK_4noneEEEE7executeERSI_iiRKSN_+18
0x0044c436         add        r7, sp, #0x8
0x0044c438         sub.w      sp, sp, #0x20c
0x0044c43c         movw       r4, #0xbbe8                                       ; :lower16:(imp___nl_symbol_ptr____stack_chk_guard - 0x44c44c)
0x0044c440         movs       r5, #0x0
0x0044c442         movt       r4, #0xe7                                         ; :upper16:(imp___nl_symbol_ptr____stack_chk_guard - 0x44c44c)
0x0044c446         movs       r0, #0x1
0x0044c448         add        r4, pc                                            ; imp___nl_symbol_ptr____stack_chk_guard
0x0044c44a         movs       r1, #0xe
0x0044c44c         ldr        r4, [r4]                                          ; imp___nl_symbol_ptr____stack_chk_guard,___stack_chk_guard
0x0044c44e         ldr        r4, [r4]                                          ; ___stack_chk_guard
0x0044c450         str        r4, [sp, #0x214 + var_C]
0x0044c452         str        r0, [sp, #0x214 + var_208]
0x0044c454         str        r1, [sp, #0x214 + var_204]
0x0044c456         str        r5, [sp, #0x214 + var_1E8]
0x0044c458         str        r0, [sp, #0x214 + var_200]
0x0044c45a         blx        imp___picsymbolstub4__getpid
0x0044c45e         str        r0, [sp, #0x214 + var_1FC]
0x0044c460         mov.w      r0, #0x1ec
0x0044c464         str        r0, [sp, #0x214 + var_20C]
0x0044c466         add        r0, sp, #0xc                                      ; argument #1 for method imp___picsymbolstub4__sysctl
0x0044c468         add        r2, sp, #0x1c                                     ; argument #3 for method imp___picsymbolstub4__sysctl
0x0044c46a         add        r3, sp, #0x8                                      ; argument #4 for method imp___picsymbolstub4__sysctl
0x0044c46c         movs       r1, #0x4                                          ; argument #2 for method imp___picsymbolstub4__sysctl
0x0044c46e         str        r5, [sp, #0x214 + var_214]                        ; argument #5 for method imp___picsymbolstub4__sysctl
0x0044c470         str        r5, [sp, #0x214 + var_210]
0x0044c472         blx        imp___picsymbolstub4__sysctl
0x0044c476         ldr        r0, [sp, #0x214 + var_1E8]
0x0044c478         ldr        r1, [sp, #0x214 + var_C]
0x0044c47a         subs       r1, r4, r1
0x0044c47c         itttt      eq
0x0044c47e         andeq      r0, r0, #0x800
0x0044c482         lsreq      r0, r0, #0xb
0x0044c484         addeq.w    sp, sp, #0x20c
0x0044c488         popeq      {r4, r5, r7, pc}
0x0044c48a         blx        imp___picsymbolstub4____stack_chk_fail
                        ; endp
0x0044c48e         nop
```

# Two more `sysctl`-based anti-debugging checks

After digging refs to `imp___picsymbolstub4__sysctl`, I found two more "interesting" procedures:

```
               __Z10IsDebuggedv:        // IsDebugged()
0x0044c490         push       {r4, r5, r7, lr}                                  ; XREF=__Z7p_beginPFvvE+1138, 0x12c84ac
0x0044c492         add        r7, sp, #0x8
0x0044c494         sub.w      sp, sp, #0x20c
0x0044c498         movw       r4, #0xbb8c                                       ; :lower16:(imp___nl_symbol_ptr____stack_chk_guard - 0x44c4a8)
0x0044c49c         movs       r5, #0x0
0x0044c49e         movt       r4, #0xe7                                         ; :upper16:(imp___nl_symbol_ptr____stack_chk_guard - 0x44c4a8)
0x0044c4a2         movs       r0, #0x1
0x0044c4a4         add        r4, pc                                            ; imp___nl_symbol_ptr____stack_chk_guard
0x0044c4a6         movs       r1, #0xe
0x0044c4a8         ldr        r4, [r4]                                          ; imp___nl_symbol_ptr____stack_chk_guard,___stack_chk_guard
0x0044c4aa         ldr        r4, [r4]                                          ; ___stack_chk_guard
0x0044c4ac         str        r4, [sp, #0x214 + var_C]
0x0044c4ae         str        r0, [sp, #0x214 + var_208]
0x0044c4b0         str        r1, [sp, #0x214 + var_204]
0x0044c4b2         str        r5, [sp, #0x214 + var_1E8]
0x0044c4b4         str        r0, [sp, #0x214 + var_200]
0x0044c4b6         blx        imp___picsymbolstub4__getpid
0x0044c4ba         str        r0, [sp, #0x214 + var_1FC]
0x0044c4bc         mov.w      r0, #0x1ec
0x0044c4c0         str        r0, [sp, #0x214 + var_20C]
0x0044c4c2         add        r0, sp, #0xc                                      ; argument #1 for method imp___picsymbolstub4__sysctl
0x0044c4c4         add        r2, sp, #0x1c                                     ; argument #3 for method imp___picsymbolstub4__sysctl
0x0044c4c6         add        r3, sp, #0x8                                      ; argument #4 for method imp___picsymbolstub4__sysctl
0x0044c4c8         movs       r1, #0x4                                          ; argument #2 for method imp___picsymbolstub4__sysctl
0x0044c4ca         str        r5, [sp, #0x214 + var_214]                        ; argument #5 for method imp___picsymbolstub4__sysctl
0x0044c4cc         str        r5, [sp, #0x214 + var_210]
0x0044c4ce         blx        imp___picsymbolstub4__sysctl
0x0044c4d2         ldr        r0, [sp, #0x214 + var_1DC]
0x0044c4d4         cmp        r0, #0x0
0x0044c4d6         it         ne
0x0044c4d8         movne      r0, #0x1
0x0044c4da         ldr        r1, [sp, #0x214 + var_C]
0x0044c4dc         subs       r1, r4, r1
0x0044c4de         itt        eq
0x0044c4e0         addeq.w    sp, sp, #0x20c
0x0044c4e4         popeq      {r4, r5, r7, pc}
0x0044c4e6         blx        imp___picsymbolstub4____stack_chk_fail
                        ; endp
0x0044c4ea         nop        
```

and

```
               _CLSProcessDebuggerAttached:
0x00bb5528         push       {r4, r5, r7, lr}                                  ; XREF=_CLSContextInitialize+148
0x00bb552a         add        r7, sp, #0x8
0x00bb552c         sub.w      sp, sp, #0x20c
0x00bb5530         movw       r4, #0x2af4                                       ; :lower16:(imp___nl_symbol_ptr____stack_chk_guard - 0xbb5540)
0x00bb5534         movs       r5, #0x0
0x00bb5536         movt       r4, #0x71                                         ; :upper16:(imp___nl_symbol_ptr____stack_chk_guard - 0xbb5540)
0x00bb553a         movs       r0, #0x1
0x00bb553c         add        r4, pc                                            ; imp___nl_symbol_ptr____stack_chk_guard
0x00bb553e         movs       r1, #0xe
0x00bb5540         ldr        r4, [r4]                                          ; imp___nl_symbol_ptr____stack_chk_guard,___stack_chk_guard
0x00bb5542         ldr        r4, [r4]                                          ; ___stack_chk_guard
0x00bb5544         str        r4, [sp, #0x214 + var_C]
0x00bb5546         str        r0, [sp, #0x214 + var_1C]
0x00bb5548         str        r1, [sp, #0x214 + var_18]
0x00bb554a         str        r5, [sp, #0x214 + var_1F8]
0x00bb554c         str        r0, [sp, #0x214 + var_14]
0x00bb554e         blx        imp___picsymbolstub4__getpid
0x00bb5552         str        r0, [sp, #0x214 + var_10]
0x00bb5554         mov.w      r0, #0x1ec
0x00bb5558         str        r0, [sp, #0x214 + var_20C]
0x00bb555a         add        r0, sp, #0x1f8                                    ; argument #1 for method imp___picsymbolstub4__sysctl
0x00bb555c         add        r2, sp, #0xc                                      ; argument #3 for method imp___picsymbolstub4__sysctl
0x00bb555e         add        r3, sp, #0x8                                      ; argument #4 for method imp___picsymbolstub4__sysctl
0x00bb5560         movs       r1, #0x4                                          ; argument #2 for method imp___picsymbolstub4__sysctl
0x00bb5562         str        r5, [sp, #0x214 + var_214]                        ; argument #5 for method imp___picsymbolstub4__sysctl
0x00bb5564         str        r5, [sp, #0x214 + var_210]
0x00bb5566         blx        imp___picsymbolstub4__sysctl
0x00bb556a         cbz        r0, 0xbb5588
	
0x00bb556c         movw       r0, #0x69c                                        ; "%s: sysctl failed while trying to get kinfo_proc\\n", :lower16:(0x1095c1c - 0xbb5580)
0x00bb5570         movt       r0, #0x4e                                         ; "%s: sysctl failed while trying to get kinfo_proc\\n", :upper16:(0x1095c1c - 0xbb5580)
0x00bb5574         movw       r1, #0x6cc                                        ; "CLSProcessDebuggerAttached", :lower16:(0x1095c4e - 0xbb5582)
0x00bb5578         movt       r1, #0x4e                                         ; "CLSProcessDebuggerAttached", :upper16:(0x1095c4e - 0xbb5582)
0x00bb557c         add        r0, pc                                            ; "%s: sysctl failed while trying to get kinfo_proc\\n", argument #1 for method _CLSSDKFileLog
0x00bb557e         add        r1, pc                                            ; "CLSProcessDebuggerAttached", argument #2 for method _CLSSDKFileLog
0x00bb5580         bl         _CLSSDKFileLog
0x00bb5584         movs       r0, #0x0
0x00bb5586         b          0xbb5592
	
0x00bb5588         ldrb.w     r0, [sp, #0x214 + var_1F7]                        ; XREF=_CLSProcessDebuggerAttached+66
0x00bb558c         and        r0, r0, #0x8
0x00bb5590         lsrs       r0, r0, #0x3
	
0x00bb5592         ldr        r1, [sp, #0x214 + var_C]                          ; XREF=_CLSProcessDebuggerAttached+94
0x00bb5594         subs       r1, r4, r1
0x00bb5596         itt        eq
0x00bb5598         addeq.w    sp, sp, #0x20c
0x00bb559c         popeq      {r4, r5, r7, pc}
0x00bb559e         blx        imp___picsymbolstub4____stack_chk_fail
                        ; endp
0x00bb55a2         nop        
```

I was not sure if that two procedures were a part of ADVobfuscator. I did not digg so deep. May be the procs were self-made.

# And, finally, `ptrace` (`svc #0x80`)-based checkpoint

Finally, I found

```
           __ZL11anti_ptracev:        // anti_ptrace()
0x00465a48         mov.w      r0, #0x1f                                         ; XREF=__Z7p_beginPFvvE+1458, __Z7p_beginPFvvE+1462, __Z7p_beginPFvvE+1466
0x00465a4c         mov.w      r1, #0x0
0x00465a50         mov.w      r2, #0x0
0x00465a54         mov.w      r3, #0x0
0x00465a58         mov.w      ip, #0x1a
0x00465a5c         svc        #0x80
```

The code is equivalent to `ptrace(PT_DENY_ATTACH, 0, 0, 0)`.

# F#$k the protection!

To disable the protection, I loaded the app under LLDB by 

```
iPhone:~ root# debugserver *:6666 -x backboard /var/mobile/Applications/BA708472-5A17-41FB-B233-1B5DA84460D6/Musical.ly.app/Musical.ly
```

Then I loaded modules and calculated the ASLR offset for `Musical.ly`. Further, I patched the checkpoints (here you see some dumps from LLDB console, the procedures' addresses are ASLR-shifted):

1. Patch `AmIBeingDebugged()`:

    ```
    (lldb) mem write 0x4b6434 0x80 0xEA 0x00 0x00 0xF7 0x46
    (lldb) dis -s 0x004b6434
    Musical.ly`AmIBeingDebugged():
        0x4b6434 <+0>:  eor.w  r0, r0, r0
        0x4b6438 <+4>:  mov    pc, lr
        0x4b643a <+6>:  ldrb   r3, [r0, #0x14]
        0x4b643c <+8>:  movw   r4, #0xbbe8
        0x4b6440 <+12>: movs   r5, #0x0
        0x4b6442 <+14>: movt   r4, #0xe7
        0x4b6446 <+18>: movs   r0, #0x1
        0x4b6448 <+20>: add    r4, pc
        0x4b644a <+22>: movs   r1, #0xe
        0x4b644c <+24>: ldr    r4, [r4]
        0x4b644e <+26>: ldr    r4, [r4]
        0x4b6450 <+28>: str    r4, [sp, #0x208]
        0x4b6452 <+30>: str    r0, [sp, #0xc]
    ```

2. Patch `IsDebugged()`

    ```
    (lldb) mem write 0x4b6490 0x80 0xEA 0x00 0x00 0xF7 0x46
    (lldb) dis -s 0x004b6490
    Musical.ly`IsDebugged():
     0x4b6490 <+0>:  eor.w  r0, r0, r0
     0x4b6494 <+4>:  mov    pc, lr
     0x4b6496 <+6>:  ldrb   r3, [r0, #0x14]
     0x4b6498 <+8>:  movw   r4, #0xbb8c
     0x4b649c <+12>: movs   r5, #0x0
     0x4b649e <+14>: movt   r4, #0xe7
     0x4b64a2 <+18>: movs   r0, #0x1
     0x4b64a4 <+20>: add    r4, pc
     0x4b64a6 <+22>: movs   r1, #0xe
     0x4b64a8 <+24>: ldr    r4, [r4]
     0x4b64aa <+26>: ldr    r4, [r4]
     0x4b64ac <+28>: str    r4, [sp, #0x208]
     0x4b64ae <+30>: str    r0, [sp, #0xc]
    ```

3. Patch `CLSProcessDebuggerAttached()`:

    ```
    (lldb) mem write 0xc1f528 0x80 0xEA 0x00 0x00 0xF7 0x46
    (lldb) dis -s 0xc1f528
    Musical.ly`CLSProcessDebuggerAttached:
        0xc1f528 <+0>:  eor.w  r0, r0, r0
        0xc1f52c <+4>:  mov    pc, lr
        0xc1f52e <+6>:  ldrb   r3, [r0, #0x14]
        0xc1f530 <+8>:  movw   r4, #0x2af4
        0xc1f534 <+12>: movs   r5, #0x0
        0xc1f536 <+14>: movt   r4, #0x71
        0xc1f53a <+18>: movs   r0, #0x1
        0xc1f53c <+20>: add    r4, pc
        0xc1f53e <+22>: movs   r1, #0xe
        0xc1f540 <+24>: ldr    r4, [r4]
        0xc1f542 <+26>: ldr    r4, [r4]
        0xc1f544 <+28>: str    r4, [sp, #0x208]
        0xc1f546 <+30>: str    r0, [sp, #0x1f8]
    ```

4. Patch `anti_ptrace()`:

    ```
    (lldb) memory write 0x4cfa48 0x70 0x47
    dis -s 0x004cfa48
    Musical.ly`anti_ptrace():
        0x4cfa48 <+0>:  bx     lr
        0x4cfa4a <+2>:  movs   r7, r3
        0x4cfa4c <+4>:  mov.w  r1, #0x0
        0x4cfa50 <+8>:  mov.w  r2, #0x0
        0x4cfa54 <+12>: mov.w  r3, #0x0
        0x4cfa58 <+16>: mov.w  r12, #0x1a
        0x4cfa5c <+20>: svc    #0x80
        0x4cfa5e <+22>: bx     lr
    ```

Finally, I run the app under debugger:

```
(lldb) c
Process 1105 resuming
2016-03-06 16:12:24.729 Musical.ly[1105:60b] SSL Kill Switch - Hook Enabled.
CoreFoundation = 847.270000
2016-03-06 16:12:28.668 Musical.ly[1105:60b][Crashlytics] Version 3.6.0 (99)
2016-03-06 16:12:29:263 Musical.ly[1105:60b] Logging: UINavigationController viewDidLoad
2016-03-06 16:12:29:396 Musical.ly[1105:60b] Logging: MLSplashTransitionViewController viewDidLoad
2016-03-06 16:12:30:186 Musical.ly[1105:60b] will open yapdatabase /var/mobile/Applications/BA708472-5A17-41FB-B233-1B5DA84460D6/Library/Application Support/db.yap
2016-03-06 16:12:30:298 Musical.ly[1105:60b] open /var/mobile/Applications/BA708472-5A17-41FB-B233-1B5DA84460D6/Library/Application Support/db.yap success
2016-03-06 16:12:31.008 Musical.ly[1105:1f13] sign time: 2016-03-07 00:12:31
[Crashlytics] The signal SIGABRT has a non-Crashlytics handler ((null)).  This will interfer with reporting.
[Crashlytics] The signal SIGBUS has a non-Crashlytics handler ((null)).  This will interfer with reporting.
 non-Crashlytics handler ((null)).  This will interfer with reporting.
[Crashlytics] The signal SIGILL has a non-Crashlytics handler ((null)).  This will interfer with reporting.
his will interfer with reporting.
[Crashlytics] The signal SIGSYS has a non-Crashlytics handler ((null)).  This will interfer with reporting.
[Crashlytics] The signal SIGTRAP has a non-Crashlytics handler ((null)).  This will interfer with reporting.
16-03-06 16:12:31:438 Musical.ly[1105:60b] Logging: UINavigationController viewDidAppear:
2016-03-06 16:12:31:443 Musical.ly[1105:60b] Logging: MLSplashTransitionViewController viewDidAppear:
ViewController viewDidLoad
2016-03-06 16:12:31:946 Musical.ly[1105:60b] Logging: MLLoginViewController viewDidLoad
2016-03-06 16:12:32:003 Musical.ly[1105:60b] Logging: MLSmoothVideoLoopViewController viewDidLoad
60b] Logging: MLWizardViewController viewDidAppear:
2016-03-06 16:12:32:780 Musical.ly[1105:60b] Logging: MLSmoothVideoLoopViewController viewDidLoad
2016-03-06 16:12:33:311 Musical.ly[1105:60b] Logging: MLLoginViewController viewDidAppear:
6:12:33:313 Musical.ly[1105:60b] Logging: MLSmoothVideoLoopViewController viewDidAppear:
2016-03-06 16:12:35:870 Musical.ly[1105:60b] Error Occured:Error Domain=org.restkit.RestKit.ErrorDomain Code=-1016 "Expected content type {(
    "text/javascript",
    "application/x-www-form-urlencoded",
    "application/json"
)}, got text/plain" UserInfo=0x14d0c430 {NSLocalizedRecoverySuggestion={'success':true}, NSErrorFailingURLKey=https://www.musical.ly/rest/v2/users/active, AFNetworkingOperationFaestErrorKeyV1=<NSMutableURLRequest: 0x16033f40> { URL: https://www.musical.ly/rest/v2/users/active }, NSLocalizedDescription=Expected content type {(
    "text/javascript",
    "application/x-www-form-urlencoded",
    "application/json"
ain, AFNetworkingOperationFailingURLResponseErrorKeyV1=<NSHTTPURLResponse: 0x14f60f30> { URL: https://www.musical.ly/rest/v2/users/active } { status code: 200, headers {
    Connection = "keep-alive";
    "Content-Length" = 16;
lication/octet-stream";
    Date = "Mon, 07 Mar 2016 00:12:35 GMT";
    Server = "nginx/1.4.6 (Ubuntu)";
} }}
2016-03-06 16:12:35:905 Musical.ly[1105:60b] Logging: MLWizardViewController viewDidAppear:
ing: MLLoginViewController viewDidAppear:
2016-03-06 16:12:36:707 Musical.ly[1105:60b] Logging: MLSmoothVideoLoopViewController viewDidAppear:
2016-03-06 16:12:37:190 Musical.ly[1105:60b] Logging: MLSmoothVideoLoopViewController viewDidDisappear:
(lldb)  
```

The app was ready for debugging :)