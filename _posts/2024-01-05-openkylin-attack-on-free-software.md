---
layout: postwtoc
title: "OpenKylin's attack on free software"
toc: true
---

Warning: there are many images (screenshots) on this page.

Normally, I don't do "distro reviews". There are many reviews online already,
and I have unsually strict requirements for something I use daily.

## The marketing

OpenKylin was dubbed "the first independent homegrown linux desktop distro"
(sic) by many state media in China, and it announced its release 1.0 in mid 2023.
This, without a doubt, is false advertising, and probably nationalist
propaganda. I had been asked by "the infrastructure team of openEuler" to host
a mirror of openEuler's OS images. After some digging, openEuler seems to be
an rpm-based distro that also has a Chinese homepage and many Chinese company
logos on it while OpenKylin is Debian-based (or more accurately,
Ubuntu-based). The two are both sponsored by the state, yet they claim to be
community projects. As of the mirroring request, I politely refused of course,
as it would not align with my philosophy, to provide better access to free
software in the world.

![openeuler-email](/assets/openkylin/openeuler-email.png)

One more distribution of GNU/Linux seems to be a good thing in general. I
expect with more choices of flavors of OSes, more people will eventually turn
to free software. [Climbing the freedom
ladder](https://www.fsf.org/blogs/community/the-journey-begins-with-a-single-step-climb-the-freedom-ladder)
has to start with somewhere afterall. However, I was soon to be proven too
naïve.

## An audit

Casually browsing the repository, I found that Sougou is set to the startup
homepage of the default browser Firefox. This was discovered in the package
openkylin-default-settings-23.05.2.

![openkylin-default-settings-23.05.2](/assets/openkylin/openkylin-default-settings-23.05.2.png)

I have read somewhere that it was just a remix of Ubuntu, but I decided to see
for myself how good or how bad this OpenKylin was.

### The download

To no one's surprise, OpenKylin's website is in Chinese by default. The title
of the page translates to "OpenKylin open source operating system". The
download is straight forward, but only MD5 checksum of the ISO image is
offered on the website, no SHA256 or GPG signature. Are we in 1985?

### The Installer and the LiveCD

Using Qemu, I could not help but notice the installer uses outdated GRUB 2.04
version.

![installer-grub](/assets/openkylin/installer-grub.png)

Booting into the LiveCD, the desktop environment looks good, but of course I
am not impressed by fancy UI. So I pressed Crtl + Alt + T to open the
terminal, and found out the rumors were true: some essential utilities are
missing. Notably the `less` from `coreutils`, GNU Nano, and `man-db`. These
are the three things I cannot live without.

![coreutils](/assets/openkylin/coreutils.png)

Poking around the desktop, I noticed a familiar "W" icon on the taskbar. It is
a Microsoft Office clone, called WPS Office. This proprietary malware indeed
is homegrown, a known offender of user freedom and privacy. [In 2022, WPS
deleted a user's file because of "sensitive
content"](https://www.scmp.com/tech/big-tech/article/3185239/chinese-word-processor-wps-accused-censorship-after-author-says-she). I
thought it was only on the LiveCD by default, but apparently I was wrong
again.

![wps](/assets/openkylin/wps.png)

Time to install. The installer asked me to accept the license
agreement. Despite I chose English language explicitly, I was presented the
license agreement in Chinese. In short, the license agreement decribes how
"we" (refers to "the OpenKylin community") collect and use user data. Remember
OpenKylin is endorsed and sponsored by the state.

![selectenglish](/assets/openkylin/selectenglish.png)

![openkylinlicenseagreement](/assets/openkylin/openkylinlicenseagreement.png)

By default, Linux kernel 5.15 and 6.1 are installed. This becomes a selling
point: the short text below translates to "innovation and stability" (whatever
that means). Both 5.15.y and 6.1.y are LTS kernels, is there any point to
install both?

![install1](/assets/openkylin/install1.png)

![install2](/assets/openkylin/install2.png)

### The first boot is the last boot

The bootloader is GRUB 2.06, the latest stable release at the time. To my
surprise, the distro's name in GRUB is called "OpenKylin GNU/Linux", exactly
on the boot menu. I am unsure if the maintainers are aware of what GNU project
is or what it represents at all.

![grub](/assets/openkylin/grub.png)

![grubadv](/assets/openkylin/grubadv.png)

The next step would be finding the usual software I need on a daily basis. If
I were a user that is not familiar with the command line, I would go to the
package manager's frontend GUI to install software. This frontend here is
called "software store". Having seen WPS Office installed by default, I did
not expect much. I was greeted with a full page of Chinese proprietary
software, many of which are known censors and malware, and it did not
recommend any free software.

![store1](/assets/openkylin/store1.png)

![store2](/assets/openkylin/store2.png)

![store3](/assets/openkylin/store3.png)

And there we are, "Weixin" a.k.a "Wechat", the most popular instant messaging
software in China, which banned my account for 24 hours in late 2022 for
discussing **sensitive content**. "Baidu Netdisk", a.k.a. "Baidu Cloud", one
of the most popular storage services in China, which permanently banned my
account in 2014 for storing **sensitive content**. Storing some pdfs and video
clips download from youtube, the next thing you know, you become an enemy of
the state.

![weixin](/assets/openkylin/weixin.png)

![baidu](/assets/openkylin/baidu.png)

![de1](/assets/openkylin/de1.png)

These proprietary software packages are are huge. There are a few theories to
explain it:

- a. they are not GNU/Linux-native software, so they use WINE

- b. everything is statically linked

- c. they include universal backdoor

- d. all of the above

The installed system uses 12GB on disk, and 1.7GB memory after booting to
desktop. What kind of bloatware is this? This is the first boot and the last
boot, I will never use this system again.

### The offensive language in documentation

There is no free software without free and good documentation. To no one's
surprise, OpenKylin fails on this too, and it fails badly. Its documentation
can be considered non-existent compared to other distros.

I discovered a few interesting things on the documentation section of the
OpenKylin website. It has a `must read for newbie` part, consisting of pages I
would consider to be blog posts rather than manuals or documentations. There
is one page titled `20 funny things about Linux commands and Linux terminal`.
It has an joke with offensive, misogynistic language in the end, as the images
shown below. The Chinese text feels like a translation, and I was not wrong.
After an easy seach online, I found a blog post in English which matches the
Chinese text.

![doc1](/assets/openkylin/doc1.png)

![doc2](/assets/openkylin/doc2.png)

![doc3](/assets/openkylin/doc3.png)

![doc4](/assets/openkylin/doc4.png)

OpenKylin's documentation pages has the CC-BY-SA 4.0 license by default, which
is a good thing *if they comply with the terms of license*, namely the *BY*
(give attribution) and *SA* (share-alike) part. Of course, the Chinese page
on OpenKylin's website does not credit to where it was adapted from, and the
original work is likely non-free.

### The reddit

A few users on Reddit say [there appears to be tons of segfaults in
dmesg](https://www.reddit.com/r/linux/comments/14zc6wn/a_quick_look_at_the_openkylin_linux_distro/),
mostly from `ukui-settings-daemon` and `kylin-status-manager`, two important
components of the desktop environment. I did not check dmesg, but I would not
be surprised. NVDIA's proprietary GPU driver can also be found in OpenKylin's
main repository.

## The verdict

**OpenKylin bundles proprietary software and installs it by default**. Users
have no choice but to accept it during the installation. **OpenKylin also
actively recommends proprietary software in its "software store"**. This is
even much worse than the [optionally
free](https://www.gnu.org/distros/optionally-free-not-enough.html) in some
other distros. It does not deserve to have "GNU" or "Linux" in its name. It
loudly rejects the ideal of Free Software Movement. In fact, it can be
considered an attack on free software in whole, and it even turns its back on
"open source" for that matter. It never cares about user or freedom at all.

I call for boycotting this distribution. The developers and maintainers with
minimum ammount of conscience that care about Chinese users' freedom should
make a 100% free distro called LibreKylin instead.

## License

Copyright 2024, Jing Luo.

This work is licensed under a [Creative Commons Attribution 4.0 International
license](https://creativecommons.org/licenses/by/4.0/).
