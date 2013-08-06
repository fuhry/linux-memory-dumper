# What is this... thing?
This is a script which dumps as much allocated memory as possible from a running Linux system. The primary target is user data, which is arguably the most difficult thing to get from a Linux box these days that is relevant to a forensic investigation.

# How does it work?
By parsing /proc/[pid]/maps and dumping the regions described in there from /proc/[pid]/mem.

# Does it need root?
No! However, the success rate is significantly more limited when you run it as an unprivileged user. I still managed to pull quite a bit of information out running it under my normal unprivileged account.

# Why?
For a forensics class, and generally because there's a lack of memory dumping utilities for Linux these days since /dev/mem was deprecated.

# Limitations

## Scope

My tool is not a replacement for /dev/mem. It dumps allocated memory from currently running user-space processes. The following are not dumped:

* Unallocated pages
* Kernel threads
* I/O cache memory
* Files mapped with mmap()
* processes which restrict the use of ptrace(2)

It is also not a one-stop magical forensic report generator. If you think otherwise, consider a different career. Alongside the use of this tool you should be collecting lots more information from the systemm, the specifics of which are outside the scope of this document.

## Technical

This script collects its list of PIDs at start-up, and then scans through each one. The process takes a while. By the time it makes it to a given process it may have died, and by the time it finishes new processes may have spawned. Since the goal of this script is to collect data from longer-running processes such as web browsers and editors, I don't consider this a huge issue. Any competent forensic examiner uses this script alongside other techniques such as disk imaging anyway.

It's slow as balls right now, I think because access to /proc is slow on my system, which is pretty up-to-date (kernel 3.8.4) probably because the kernel generates maps files and such on the fly.

It probably will not obtain dm-crypt encryption keys, as those are likely to be stored in the memory of kernel threads, but those can easily be obtained (if you have root) with "dmsetup table --showkeys".

Everything about this script is also based on virtual memory addresses, not physical. This will complicate the process of verifying results. Memory imaging in general is, in my opinion, very difficult to verify because RAM by its very nature is constantly changing, and it is impossible to image a box's memory without changing it in some way. Thus, the ability of this tool's output to stand up in court is questionable.

This tool is not intended to image every byte on the box, and is incapable of doing so. This applies in particular to free (unallocated) and I/O cache memory.

# Inhibition

The good folks on /r/netsec have proposed compiling your kernel with CONFIG_GRKERNSEC_PROC, CONFIG_GRKERNSEC_PROC_MEMMAP, and CONFIG_GRKERNSEC_HARDEN_PTRACE.

# Disclaimer
I've written this utility in good faith. It's open source, you can see exactly how it gathers its data, and you can go back to the Linux kernel docs or even the kernel source to examine its behavior. All this said, I cannot and will not be held liable for any inaccurate results returned by this tool, nor is any warranty provided (please see the license below).

Also, if you are a forensic examiner, please do not exclude the possibility that a smart enemy/target has modified their kernel to lie about memory regions or lie in /proc/[pid]/mem. It's pretty easy to do.

# License
The MIT License (MIT)

Copyright (c) 2013 Dan Fuhry <dan@fuhry.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
