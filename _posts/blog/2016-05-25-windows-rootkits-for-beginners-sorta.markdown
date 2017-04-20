---
layout: post
title: Windows Rootkits for Beginners (sorta)
categories: blog
excerpt: The first part of rootkits is the injection...
tags: [reverse engineering, security, malware]
image:
    feature:
date: 2016-05-25T19:12:05-07:00
---
It's windows rootkits day today.

Rootkits can be of several types depending on the privilege ring in which they operate, ranging from user mode (Ring 3) all the way down to firmware and hardware based. Let’s stick with user mode for now. This kind manipulates code residing in userspace.

Rootkitting (as used here) is the process by which an evil process will modify the functionality of an existing process to create secret accounts, hide processes/files/network connections/events from it etc. To accomplish this, a cutesy-wootsey rootkit has to take two steps:

* DLL injection— which is the part where it “injects” malicious code into a DLL residing in that process’s user (memory) space. A few known ways of doing this:-

    * SetWindowsHookEx function

    * CreateRemoteThread + LoadLibrary— Typically, here’s what you do. Get a handle to the target process (create it, or query all the running processes and scan for the target executable’s filename.) Then allocate some memory in the process and write the name of the DLL to be injected to it. This is usually done using the VirtualAllocEx function call followed by a WriteProcessMemory call. Then you would create a remote thread in the process, setting its start address to that of LoadLibrary with its argument as the name of the DLL you just wrote in the target process.

    * There are a bunch of other ways to do DLL injection— using SuspendThread, or NTSuspendThread, using system-level shims etc. For future posts.
* API Hooking — which is the part that watches for the sys_calls pertaining to the functionality the rootkit wants to modify and redirects those calls to the rootkit code to do with it what it may


So I have a tool called kinject.exe (details here— <a href="https://www.aldeid.com/wiki/KInject">https://www.aldeid.com/wiki/KInject</a>). It is supposed to inject an arbit DLL into and arbit process. I loaded it into ollydbg. It needs a few arguments— the name of the process to inject in, the path to the DLL to be injected, and a “—create” argument if the process is not already running (I used notepad.exe and —create). Once loaded in olly, using ctrl+n to bring up the names window, I see all the expected system calls. CreateProcessA, OpenProcess (incase I want to attack one that is already running) (either of those to get a handle of the process I want to inject in), VirtualAllocEx (to allocate memory in the remote process), WriteProcessMemory (to write the name of the bad DLL in the memory I just allocated), GetProcAddress (to find the address of the LoadLibraryA DLL in user space), and CreateRemoteThread (which triggers the call to LoadLibraryA with the malicious DLL as argument). I put breakpoints at all these functions to see each step of the process.

<figure>
    <a href="/images/post4-image1.png"><img src="/images/post4-image1.png" alt="image"></a>
</figure>

This is where the notepad process gets run. Note the arguments in the stack. The notepad window popped up right after I executed that statement.

<figure>
    <a href="/images/post4-image2.png"><img src="/images/post4-image2.png" alt="image"></a>
</figure>

Stack set up for VirtualAllocEx.

<figure>
    <a href="/images/post4-image3.png"><img src="/images/post4-image3.png" alt="image"></a>
</figure>

The virtual memory was allocated here. Note the return value in EAX. This is the base address allocated in the victim process (here, notepad) where we will write the name of the malicious DLL.

<figure>
    <a href="/images/post4-image4.png"><img src="/images/post4-image4.png" alt="image"></a>
</figure>

Note that the BaseAddress argument in the stack is the same as what was returned by VirtualAllocEx. The address of the buffer provided as argument (see stack) is also seen in the hexdump view. We can see that it contains the string referring to the DLL we want to inject. Sweet!

<figure>
    <a href="/images/post4-image5.png"><img src="/images/post4-image5.png" alt="image"></a>
</figure>

This is the setup for the GetProcAddress call. Note that the GetModuleA call was first used to get a handle to Kernel32.DLL (which contains LoadLibraryA). The handle and the name of the function whose address is needed are the arguments to GetProcAddress.

<figure>
    <a href="/images/post4-image6.png"><img src="/images/post4-image6.png" alt="image"></a>
</figure>

You can see that EAX contains the handle to Kernel32.DLL.

<figure>
    <a href="/images/post4-image7.png"><img src="/images/post4-image7.png" alt="image"></a>
</figure>

GetProcAddress used to get the address of LoadLibraryA.

<figure>
    <a href="/images/post4-image8.png"><img src="/images/post4-image8.png" alt="image"></a>
</figure>

Here you can see the address of LoadLibraryA in the userspace is returned in EAX. This will be used in our CreateRemoteThread Call.

<figure>
    <a href="/images/post4-image9.png"><img src="/images/post4-image9.png" alt="image"></a>
</figure>

Check out the stack which contains the arguments for the CreateRemoteThread call. The hRemoteProcess argument refers to notepad, the start address is that of LoadLibraryA that we got from the getProcAddress call. And the parameter to be passed to LoadLibraryA is the address where we wrote "C:\kinject\kntillusion.dll" -- the name of the dll we want to load.

<figure>
    <a href="/images/post4-image10.png"><img src="/images/post4-image10.png" alt="image"></a>
</figure>

And viola, the injected DLL can be seen in the modules for notepad in processHacker.

This of course, is only the first step of the rootkitting process. Next is the modification of functionality.

Bibliography:

- Windows Rootkit Overview by Symantec: <a href="https://www.symantec.com/avcenter/reference/windows.rootkit.overview.pdf">https://www.symantec.com/avcenter/reference/windows.rootkit.overview.pdf</a>
- Wikipedia Article on Rootkits: <a href="https://en.wikipedia.org/wiki/Rootkit">https://en.wikipedia.org/wiki/Rootkit</a>
- Nice paper explaining rootkits and how to use the hackerDefender rootkit: <a href="http://www.carnal0wnage.com/papers/rootkit_for_the_masses.pdf">http://www.carnal0wnage.com/papers/rootkit_for_the_masses.pdf</a>
