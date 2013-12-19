---
layout: default
title: My Hardest Bug
---

> What's the hardest bug you've ever debugged?

Every programmer has their heroically exaggerated tales of death defying combat
and daring beast slaying. Here's my epic tale.


Memory Corruption
-----------------

Not six months out of school, fresh diploma in hand, I'm working in an embedded
Linux firmware development role at Lexmark chasing a memory corruption problem.
Memory corruption tends to be difficult to debug since the symptoms are often
far removed from the cause. Maybe there's a buffer overflow, an uninitialized
pointer, a double free, or an errant [DMA][] somewhere, but all you see are
mysterious problems: hangs, undefined instructions, print defects, and
unhandled kernel faults. All too often, memory corruption is seemingly random
and impossible to reproduce consistently.

  [DMA]: http://en.wikipedia.org/wiki/Direct_memory_access

The first step to debugging these sorts of problems is to identify a
reproducible scenario if at all possible. This story starts getting interesting
after we miraculously found one.

At the time, we would see crashes caused by memory corruption once every few
hundred hours of run time. Then, one day, someone found a particular print job
that would cause a memory corruption related crash within minutes. Why that
particular print job caused the problem, I'll never know. Now, we had something
to go on.


Debugging
---------

Armed with a reproducible scenario, I started looking for patterns in the
crashes. The most enlightening were undefined instruction kernel panics which
would happen maybe a third of the time. The address of the undefined
instruction would be a reasonable kernel code address, but the instruction read
by the CPU would not be the instruction that was supposed to be there. Simple
enough, maybe someone accidentally wrote over the instructions. With a few
`printk`s added to the undefined instruction handler, I could see what the
memory near the faulting instruction looked like.

After a number of unsuccessful attempts to wean more data out of these crashes,
one particular crash looked promising.


One Crash to Rule them All
--------------------------

This crash would unravel the mystery. We were using a dual core CPU at the
time. In this particular crash, first CPU1 received an unhandled kernel fault
in a valid module address range while trying to execute module code (perhaps
indicating a corrupted [page table][] or invalid [TLB][]). While this error was
being processed, CPU0 received an illegal instruction trap in core kernel
address space. The illegal instruction was the more interesting of the two
crashes.

  [page table]: http://en.wikipedia.org/wiki/Page_table
  [TLB]: http://en.wikipedia.org/wiki/Translation_lookaside_buffer

Here is an excerpt of the data that was printed from the modified undefined
instruction handler (translated to physical addresses, faulting address is in
parenthesis):

    undefined instruction: pc=0018abc4
    0018aba0: e7d031a2 e1b03003 1a00000e e2822008
    0018abb0: e1520001 3afffff9 e1a00001 e1a0f00e
    0018abc0: 0bd841e6 (ceb3401c) 00000004 00000001
    0018abd0: 0d066010 5439541b 49fa30e7 c0049ab8
    0018abe0: e2822001 eafffff1 e2630000 e0033000
    0018abf0: e16f3f13 e263301f e0820003 e1510000

Here is what that region of memory should have looked like:

    0018aba0: e7d031a2 e1b03003 1a00000e e2822008
    0018abb0: e1520001 3afffff9 e1a00001 e1a0f00e
    0018abc0: e3310000 (0afffffb) e212c007 0afffff3
    0018abd0: e7d031a2 e1b03c33 1a000002 e3822007
    0018abe0: e2822001 eafffff1 e2630000 e0033000
    0018abf0: e16f3f13 e263301f e0820003 e1510000

Exactly one cache line (the 32 bytes in the middle) of data was corrupted. A
co-worker identified the word `0x49fa30e7` (in the corrupted cache line) as a
magic cookie that marked the beginning of an entry into a particular ring
buffer in the system. The last word in an entry was always a timestamp, so
`0x5439541b` was the timestamp for the previous entry. I was now determined to
get the contents of that ring buffer off the machine, but it had now hung on an
inoperable [KGDB][] prompt. The machine was as good as dead.

  [KGDB]: http://en.wikipedia.org/wiki/KGDB


Cold Boot Attack
----------------

To get the contents of the ring buffer, I performed a [cold boot attack][]. I
dug up a copy of the schematic for the board I was using and found an
unpopulated pad connected to the CPU reset line. I shorted that, resetting the
CPU, but leaving DRAM intact. Then, I halted boot in the boot loader.

  [cold boot attack]: http://en.wikipedia.org/wiki/Cold_boot_attack

From the boot loader, I dumped the contents of the ring buffer in question.
Thankfully, the buffer was always located at a fixed physical address, so
finding it wasn't a problem.

Analyzing the ring buffer near the errant timestamp revealed two stale cache
lines. They contained valid data, but the timestamps in those two cache lines
were from the previous time through the ring buffer.

The corrupted cache line that caused the undefined instruction on CPU0 neatly
fit in one of the two stale cache lines in the ring buffer, but did not make
sense in the other possible position. I found a [smoking gun][]. Presumably,
the other missing cache line caused the unhandled kernel fault on CPU1.

  [smoking gun]: http://en.wikipedia.org/wiki/Smoking_gun


Misplaced Cache Lines
---------------------

The cache line should have been written to `0x0ebd2bc0` (the stale line in the
ring buffer), but was instead written to `0x0018abc0` (the corrupted kernel
code). These addresses belonged to the same [cache set][] on our CPU as bits
`[14:5]` match. The cache lines had aliased somehow.

  [cache set]: http://en.wikipedia.org/wiki/CPU_cache

                        bit   28   24   20   16   12    8    4    0
                               |    |    |    |    |    |    |    |
    0x0ebd2bc0 in binary is 0000 1110 1011 1101 0010 1011 1100 0000
    0x0018abc0 in binary is 0000 0000 0001 1000 1010 1011 1100 0000

The bottom 5 bits of an address are an index into a cache line (32 byte cache
lines). The next 10 bits, `[14:5]`, identify the cache set. The remaining bits,
`[31: 15]`, are used as an tag to identify which cache line is currently stored
in the cache.

I submitted a bug report to our CPU vendor and they identified a work-around
and it got fixed in the next revision of the CPU.

I look forward to hearing more tall tales and collecting a few more myself.
