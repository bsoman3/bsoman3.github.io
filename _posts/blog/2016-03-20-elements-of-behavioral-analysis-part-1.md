---
layout: post
title: "Elements of Behavioral Analysis - Part 1"
exerpt: "Reverse All Things"
modified:
categories: blog
tags: [reverse engineering, security, malware]
image:
  feature:
date: 2016-03-20T22:44:28-07:00
---
Here begins a month long initiative to REVERSE ALL THINGS!! and kind of keep myself accountable to it (and also, get better at this).

The sample that I looked at was brbbot. I won’t do a full behavior analysis of this (cause lets face it, who cares about that) (for those who are interested (after all these years? really?), it can be found <a href=" http://andrewjkerr.com/assets/documents/CIS4930Practical2Writeup.pdf">here</a> (also, google!). Because, as sarcastic as I can’t help sounding, I really want you to love me!!).

Anyhoo, I’m going to slice the data a bit differently (and hopefully usefully) for those who want some insights into the ideas behind reversing.

So behavioral analysis can be broken down into behavior on the end-point, and behavior on the network. In this post I will (mostly) talk about “behavior” on the end point (couldn’t resist the quotes. I have already said behavior so many times in this post, it is already feeling like “cyber” to me)

End point behavior can be broken into:

- Process action
    - Which are the processes running, process tree, image path, user and privilege with which it is running, command used and arguments (if any)
    - Threads starting/ending
    - DLLs and device drivers loaded
    - System calls made
    - Mutexes created, etc.
- Registry action
    - Which registry keys are created, deleted, queried and read.
- File system action
    - Which files are being created, deleted, written or read on file systems local and remote.

To observe all this stuff, I ran 3 tools on a vm running brbbot— CaptureBAT, ProcessMonitor and RegShot.

Reasons for using multiple tools:

- Help reinforce each other’s findings
- Show different detail aspects for same system action. For example, process monitor has the timing and process information for events, regshot has the registry key details, and CaptureBAT gets the network traffic.

What did brbbot do?
With our trusty tools, we got a bunch of noise, but were able to see the following things of interest.

It tried to copy itself to the system32 directory, but was denied access.
<figure>
    <a href="/images/system32 access denied.png"><img src="/images/system32 access denied.png" alt="image"></a>
</figure>

It also created an encrypted config file brbbot.tmp

<figure>
    <a href="/images/brbbot.tmp.png"><img src="/images/brbbot.tmp.png" alt="image"></a>
</figure>

Other analyses of this malware have found evidence that it adds itself to the  HKLM\Software\Microsoft\Windows\CurrentVersion\Run registry to establish its persistence. I did not find this in my analysis. So I was curious if this sample did anything else to establish persistence.

<a href="http://www.forensicmag.com/articles/2012/05/windows-7-registry-forensics-intrusion-related-activities">This article</a> enumerates some registry keys that can be used to establish persistence. I did not find any of these to be used in this case to establish persistence. So that remains a mystery for now.

I did find 2 things of interest from just a forensics perspective though. 

- HKCU\Software\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Compatibility Assistant\Store: For Windows 8, whenever a third party program is run, its path is added to this key. Before windows 8, this was done only for third party programs that had an installation routine, and the "Persisted" key instead of "Store" was used.
- Amcache.Hve: Windows 8 has this hive file which replaces "RecentFileCache.bcf" from earlier versions. I carries a record of every recent file run and provides information including executable path, created and modified timestamps of the file, its sha1 hash, its PE linker time stamp, and some PE header information.

Both these elements could be used to check for recently run processes by malware, rookits and what not.

That's it for today folks! Tune in again tomorrow night, same time, same channel! Until then!

###### Update:
So, I mucked up a bit. The binary didnt copy itself to System32 because I didnt run the malware with admin perms in my VM. Sorry. Once I did that, things fell into place. It also established persistence as advertised else where. Here is the autoruns snapshot that shows it.

<figure>
    <a href="/images/autoruns1.png"><img src="/images/autoruns1.png" alt="image"></a>
</figure>

Bibliography:

- What exactly does process monitor do: <a href="http://www.howtogeek.com/school/sysinternals-pro/lesson4/all/">http://www.howtogeek.com/school/sysinternals-pro/lesson4/all/</a>
- Using Process Monitor: <a href="https://www.k-state.edu/its/security/training/2009-4-9/presentations/handouts/Process_Monitor_Tutorial_Handout.pdf">https://www.k-state.edu/its/security/training/2009-4-9/presentations/handouts/Process_Monitor_Tutorial_Handout.pdf</a>
- Using process monitor to identify possible code injection: <a href="http://espionageware.blogspot.com/2014/11/using-process-monitor-to-identify_12.html">http://espionageware.blogspot.com/2014/11/using-process-monitor-to-identify_12.html</a>
- Details on windows registry keys, Part 1: <a href="https://blog.cylance.com/windows-registry-persistence-part-1-introduction-attack-phases-and-windows-services">https://blog.cylance.com/windows-registry-persistence-part-1-introduction-attack-phases-and-windows-services</a>
- Details on windows registry keys Part 2: <a href="https://blog.cylance.com/windows-registry-persistence-part-2-the-run-keys-and-search-order">https://blog.cylance.com/windows-registry-persistence-part-2-the-run-keys-and-search-order</a>
- AppCompat Keys: <a href="http://journeyintoir.blogspot.com/2013/12/revealing-program-compatibility.html">http://journeyintoir.blogspot.com/2013/12/revealing-program-compatibility.html</a>
- Amcache.hve: <a href="http://www.swiftforensics.com/2013/12/amcachehve-in-windows-8-goldmine-for.html">http://www.swiftforensics.com/2013/12/amcachehve-in-windows-8-goldmine-for.html</a>
- Registry Keys related to intrusion: <a href="http://www.forensicmag.com/articles/2012/05/windows-7-registry-forensics-intrusion-related-activities">http://www.forensicmag.com/articles/2012/05/windows-7-registry-forensics-intrusion-related-activities</a>
