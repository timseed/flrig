# IC7000 crashing on flrig

I have tried to use FLRIG several times to control my old IC7000, but it just does not seem a reliable mechanism. Thinking this was a CIV issue, I rebuilt the CIV cable several times - and also installed on another OS (Windows and Rasberry Pi) as well as my usual Mac - same result.... At the INIT phase, it crashes.

I should point out that I can control the IC7000 via the rigctl interface, so I am sure it is receiving the commands - the settings I use are like this 

```bash
rigctld -m 360  -r /dev/<MY USB Device for the IC7000>  -s 9600  --port=4532
```

I then verify that the radio is working by using NetCat (nc) 

```bash
echo "f" | nv 127.0.0.1 4532 
```

And I see the VFO-A Frequency.

If you can not get this to work, you need to **STOP** and get this working first. 


## Source Code time

I downloaded the FLRIG 1.3.51, and after building and installing FLTK (Mac and RasberyyPi) - I built the Flrig - all very easy and straight forward.

In case you have not done this before you will need to get the source code, and then compile it... it willgo something like this


### Compiling

```bash
cd ~/Dev
wget http://www.w1hkj.com/files/flrig/flrig-1.3.51.tar.gz
tar -zxvf flrig-1.3.51.tar.gz
cd flrig-1.3.51
./configure --enable-threads 
```

You may get a warning that fltk, is required.... in which case .... 

```bash
cd ~/Dev
wget https://www.fltk.org/pub/fltk/1.3.5/fltk-1.3.5-source.tar.gz
tar -zxvf fltk-1.3.5-source.tar.gz 
cd fltk-1.3.5
./configure --enable-threads
make -j4
sudo make install
```

So assuming we now have fltk installed .... we need to return to building flrig...

```
cd ~/Dev/flrig-1.3.51
./configure --enable-debug --prefix="$HOME/flrig"
make -j4
make install
```

Please note: This has installed a local copy into your $HOME/flrig area. This is intentional.


### Running

Please put your IC7000, into FM Mode....  If it is in SSB or CW or RTTY it will not crash - just FM (and AM).


Lets make sure that your copy behaves the same as the Downloaded software... so just run it ....

Mine crashed  - I hope that yours did also.

### Debug First steps

I am running on a Mac, but the Mac Debugger and the Linux debugger and very similar, so you should be able to translate between the two.

Lets load the image in the debugger and see where we get too.

```bash
lldb $HOME/flrig/bin/flrig
```

We now need to tell the debugger to run/start/execute 

So we type r and press return

```bash
r
```

Flrig should now run, and (hopefully) crash at some place ...

The output I see is this


```text
Process 11868 stopped
* thread #1, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x600000000)
    frame #0: 0x00007fff71fa4e52 libsystem_platform.dylib`_platform_strlen + 18
libsystem_platform.dylib`_platform_strlen:
->  0x7fff71fa4e52 <+18>: pcmpeqb (%rdi), %xmm0
    0x7fff71fa4e56 <+22>: pmovmskb %xmm0, %esi
    0x7fff71fa4e5a <+26>: andq   $0xf, %rcx
    0x7fff71fa4e5e <+30>: orq    $-0x1, %rax
Target 0: (flrig) stopped.
```

Ummm.... I wonder where we are .... 



#### backtrace or Show Stack 

Although the program has crashed, there is still vaaluable data in the stack - we issue the command **bt**

```
bt
```

And again I see the following

```text
* thread #1, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x600000000)
  * frame #0: 0x00007fff71fa4e52 libsystem_platform.dylib`_platform_strlen + 18
    frame #1: 0x00007fff71e40891 libsystem_c.dylib`__vfprintf + 5379
    frame #2: 0x00007fff71e66ad3 libsystem_c.dylib`__v2printf + 475
    frame #3: 0x00007fff71e4cee7 libsystem_c.dylib`_vsnprintf + 417
    frame #4: 0x00007fff71e4cf46 libsystem_c.dylib`vsnprintf_l + 41
    frame #5: 0x00007fff71e3df25 libsystem_c.dylib`snprintf + 156
    frame #6: 0x0000000100279ffb flrig`print(data=0x000000010044c6f0) at support.cxx:188:2
    frame #7: 0x0000000100289887 flrig`read_vfoA_vals() at support.cxx:3241:29
    frame #8: 0x0000000100289bf2 flrig`read_rig_vals() at support.cxx:3310:3
    frame #9: 0x000000010028fc95 flrig`init_generic_rig() at support.cxx:4411:3
    frame #10: 0x0000000100295301 flrig`initRig() at support.cxx:5643:4
    frame #11: 0x0000000100296515 flrig`initStatusConfigDialog() at support.cxx:5847:2
    frame #12: 0x0000000100035591 flrig`startup((null)=0x0000000000000000) at main.cxx:289:2
    frame #13: 0x000000010031441e flrig`do_timer(__CFRunLoopTimer*, void*) + 47
    frame #14: 0x00007fff37d5e7f9 CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__ + 20
    frame #15: 0x00007fff37d5e35f CoreFoundation`__CFRunLoopDoTimer + 859
    frame #16: 0x00007fff37d5de47 CoreFoundation`__CFRunLoopDoTimers + 322
    frame #17: 0x00007fff37d42bea CoreFoundation`__CFRunLoopRun + 1871
    frame #18: 0x00007fff37d41e3e CoreFoundation`CFRunLoopRunSpecific + 462
    frame #19: 0x00007fff3696eabd HIToolbox`RunCurrentEventLoopInMode + 292
    frame #20: 0x00007fff3696e6f4 HIToolbox`ReceiveNextEventCommon + 359
    frame #21: 0x00007fff3696e579 HIToolbox`_BlockUntilNextEventMatchingListInModeWithFilter + 64
    frame #22: 0x00007fff34fb4039 AppKit`_DPSNextEvent + 883
    frame #23: 0x00007fff34fb2880 AppKit`-[NSApplication(NSEvent) _nextEventMatchingEventMask:untilDate:inMode:dequeue:] + 1352
    frame #24: 0x0000000100314f9c flrig`fl_wait(double) + 150
    frame #25: 0x0000000100315088 flrig`fl_mac_flush_and_wait(double) + 179
    frame #26: 0x000000010031fe6b flrig`Fl::run() + 27
    frame #27: 0x000000010003719a flrig`main(argc=1, argv=0x00007ffeefbff8e0) at main.cxx:538:9
    frame #28: 0x00007fff71daecc9 libdyld.dylib`start + 1
```

That seems a lot of data - but let's just look at this for a second ....

This is the Stack, what has been called, and then what it calls..... The problem will be somewhere close to the top.

Notice how frame #6 is from code that we compiled... (the lines before this are System objects... and so are implementing something we are asking for).

```text
 frame #6: 0x0000000100279ffb flrig`print(data=0x000000010044c6f0) at support.cxx:188:2
```

It shows a filename, and the line (188).

I quit this debug session by typing **q** and pressing return.


## Debug Exploring

We will go back into the Debugger and stop just before where the program crashed. Note the crash is in **support.cxx** and at line **188**.... So I will try and "land" just before that point.

To make the code a little easier to see on screen I will ask that the debugger shows me more than 8 lines

     source list -c 40 -l 88

So the 2nd debug session will look like this

```
lldb  ~/flrig/bin/flrig
b support.cxx:185
source list -c 40 -l 88
r
```

So the program has paused, and not crashed.....

I can move forward by typing **n** (Next line) or **ni** (next instruction)

And I now see a very large line starting *snprintf(* appearing... as I press **n** - this causes a crash.

I now know exactly which line has gone wrong.

## Resolving

You need to analyze the data which the snprintf is trying to produce, and in doing so, it aappears that there is a problem with the bwt (Band Width Table).

### dsphi and dsplo 

These are inherited from RIG_IC746PRO, and the IC7000 is based on that class.

The Issue appears to be on the BandWidth setting ... 
in suuport.cxx we have a line of code

    (bwt ? bwt[data.iBW] : "n/a")a

The array bwt was populated from a call which ends up in src/rigs/IC746.cxx called bwtable

As my IC7000 is in fm more, this mode value is 5, which results in the AM table being returned (??) this array/list
looks like 

    ([0] = "FILT-1", [1] = "FILT-2", [2] = "FILT-3", [3] = 0x0000000000000000)


But as the value of data.iBW is 28, the attempt to carry out bwt[28] when there are only 4 elements is a memory violation, and something gets upset - Segmentation error, and then ..... the party is over. 

At this point you have successfully debugged, and found the issue.
