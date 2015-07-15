---
layout: post
title: How to disable Hyper-V on Windows 8
---

I like the Hyper-V technology on Windows 8 - it comes for free with the operating system, so you can run your VMs etc etc.

Unfortunately it is so deeply embedded that once you set it up you are not able to remove it any longer (at all).
I read that this is a known bug in Windows 8 (NOT 8.1).

One funny side-effect of this is that you cannot use other VM managers such as VMWare or Intel hardware acceleration for Android x86 VMs.
If Hyper-V is on, it disables the usage of VT-x and corresponding technologies.

This situation was a real problem for me, because I was not going to do a fresh Win 8.1 installation
(this is another story - if you have a non-standard layout of the ProgramData folder, eg on a non-system drive, as I have - you cannot _upgrade_ your Windows).

Fortunately, I have found a solution to this problem (thanks Scott Hanselman) - you can disable Hyper-V during OS boot.
So you can just create a new boot configuration with Hyper-V disabled - and after that you can run any virtualization environments that you have :).

Details (guid will be different):
```
C:\>bcdedit /copy {current} /d "No Hyper-V"
The entry was successfully copied to {ff-23-113-824e-5c5144ea}.

C:\>bcdedit /set {ff-23-113-824e-5c5144ea} hypervisorlaunchtype off
The operation completed successfully.
```
Then you need to reboot holding down SHIFT on the keyboard.
