---
layout: post
title: Elements of Behavioral Analysis - Part 2
modified:
categories: blog
excerpt: The frustration
tags: [reverse engineering, security, malware]
image:
  feature:
date: 2016-03-25T17:41:48-07:00
---

The challenge with learning to reverse stuff (if you're like me, going through books, online courses etc.) is this -- the course or book will give you a tamed binary. And you can use the techniques/steps laid out in the chapter/video and you will see the magic happen as planned. Things will decrypt, needles in the haystack will be found as soon as you hit enter, all the secrets will be revealed. But when you want to have some fun of your own with it, before you know it you will go down a deep well of non-ascII chars, IDA views, and assembly code that makes no sense, dream about them and maybe (if you have some sanity left) walk away to lick your wounds.

This is precisely what happened to me. I was minding my own business reversing brbbot as per instructions, and everything was dandy. I was able to decrypt the config file like so :

<figure>
    <a href="/images/decryptconfigtmp.png"><img src="/images/decryptconfigtmp.png" alt="image"></a>
</figure>

I used the encode key from here to decrypt the network traffic (which was captured using fakedns and just an httpd service).

<figure>
    <a href="/images/httpd1.png"><img src="/images/httpd1.png" alt="image"></a>
</figure>

<figure>
    <a href="/images/decrypted network traffic.png"><img src="/images/decrypted network traffic.png" alt="image"></a>
</figure>

Even messed a little bit with the C2 functionality of the bot (that's why I'm learning this stuff right?!)

If you, 

{% highlight css %}
remnux@remnux:/var/www$ echo "cexe c:\Windows\notepad.exe" > ads.php 
{% endhighlight %}

the bot will pick up the command when it does its regular web request. (Again, detailed instuctions are available <a href=" http://andrewjkerr.com/assets/documents/CIS4930Practical2Writeup.pdf">here</a>)

<figure>
    <a href="/images/runningnotepad.png"><img src="/images/runningnotepad.png" alt="image"></a>
</figure>

I could do the same thing with the tixe/exit command. But that is where I hit the limits of my RE-funhouse. I tried

{% highlight css %}
remnux@remnux:/var/www$ echo "fnoc uri=ren.php;exec=cexe;file=elif;conf=fnoc;exit=tixe;encode=5b;sleep=30000" >ads.php
{% endhighlight %}

but that did not work. I thought I could dive into Ollydbg and reverse the format required to change the config (coz THAT is the point of this!).

Suffice to say that I spun my wheels at ollyDbg for about 4 hours before acknowledging that this is probably the limit of my reversing skills today. Or I need to switch to IDA for a bit and see if it can help me do more.

Hate this feeling.

Until next time!
