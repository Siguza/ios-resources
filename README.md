# iOS Hacking Resources

## Basics

Official references:

- [ARMv8 Instruction Set Overview](https://www.element14.com/community/servlet/JiveServlet/previewBody/41836-102-1-229511/ARM.Reference_Manual.pdf) (short, kinda outdated at this point)
- [ARMv8 Architecture Reference Manual](https://developer.arm.com/docs/ddi0487/latest) (long)
- [ARM A-Profile Exploration tools](https://developer.arm.com/products/architecture/cpu-architecture/a-profile/exploration-tools) (same as above, but in machine readable form)
- [ARM System Architecture Software Standards](https://developer.arm.com/architectures/system-architectures/software-standards) (ABIs, extensions, etc.)

My own doing:

- [arm64 assembly crash course](https://github.com/Siguza/ios-resources/blob/master/bits/arm64.md)
<!-- TODO: something about memory regions and access permissions -->
<!-- TODO: something about C++ vtables -->
<!-- TODO: something about symbol stubs -->

Note on ARM documents:

> Both infocenter.arm.com and developer.arm.com are outright nightmares to navigate, and search engines don't help either. But if you have any ARM document as a PDF and want to check for a newer version, there is a neat trick. At the bottom of any page of the PDF, you should have a document identifier like so:
>
> ![Screenshot](https://user-images.githubusercontent.com/1659374/60986368-9cc60100-a33f-11e9-8ee6-b7dd89f0231e.png)
>
> That should have the form `ARM XXX ddddX.x`. Take the three letters and following four digits (in this case, `DDI0406`) and construct an URL like so:  
> `https://developer.arm.com/docs/XXXdddd/latest`  
> (In this case, `https://developer.arm.com/docs/DDI0406/latest`.)

## Internals

**Mach-O**

- Jonathan Levin - [DYLD DetaYLeD](http://www.newosxbook.com/articles/DYLD.html) <!-- Aug 2013 -->
- Jonathan Levin - [Code Signing](http://www.newosxbook.com/articles/CodeSigning.pdf) <!-- April 2015 -->

**Sandbox**

- Jonathan Levin - The Apple Sandbox ([Video](https://youtu.be/mG715HcDgO8) and [Slides](http://newosxbook.com/files/HITSB.pdf)) <!-- Sep 2016 -->

**IPC**

- Apple - Mach ([Overview](https://developer.apple.com/library/content/documentation/Darwin/Conceptual/KernelProgramming/Mach/Mach.html) and API documentation (inside the [XNU source](https://opensource.apple.com/tarballs/xnu/) in `osfmk/man/index.html`))
- nemo - [Mach and MIG](https://www.exploit-db.com/papers/13176/) (examples are outdated and for PPC/Intel, but descriptions are still accurate) <!-- 2006 -->
- Ian Beer - Apple IPC ([Video](https://vimeo.com/127859750) and [Slides](https://thecyberwire.com/events/docs/IanBeer_JSS_Slides.pdf)) <!-- May 2015 -->

**File Systems**

- Apple - [APFS Reference](https://developer.apple.com/support/apple-file-system/Apple-File-System-Reference.pdf)

**Kernel**

- Apple - [Kernel Programming Guide](https://developer.apple.com/library/content/documentation/Darwin/Conceptual/KernelProgramming)
- Apple - IOKit Fundamentals (available as [Website](https://developer.apple.com/library/content/documentation/DeviceDrivers/Conceptual/IOKitFundamentals) or [PDF](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.693.3915&rep=rep1&type=pdf))
- Apple - [About the Virtual Memory System](https://developer.apple.com/library/content/documentation/Performance/Conceptual/ManagingMemory/Articles/AboutMemory.html)
- qwertyoruiopz - Attacking XNU (Part [One](https://web.archive.org/web/20160131061526/http://blog.qwertyoruiop.com/?p=38) and [Two](https://web.archive.org/web/20160131061526/http://blog.qwertyoruiop.com/?p=48)) <!-- July 2015 -->
- Stefan Esser - [Kernel Heap](http://gsec.hitb.org/materials/sg2016/D2%20-%20Stefan%20Esser%20-%20iOS%2010%20Kernel%20Heap%20Revisited.pdf) (I hope I don't get sued) <!-- Aug 2016 -->

**Kernel Integrity**

- xerub - [Tick Tock](https://xerub.github.io/ios/kpp/2017/04/13/tick-tock.html)
- Siguza - [KTRR](https://siguza.github.io/KTRR/)
<!-- TODO: something about PPL/APRR -->

**Control Flow Integrity**

- Brandon Azad - [Examining Pointer Authentication on the iPhone XS](https://googleprojectzero.blogspot.com/2019/02/examining-pointer-authentication-on.html)
- Qualcomm Product Security - [Pointer Authentication on ARMv8.3](https://www.qualcomm.com/media/documents/files/whitepaper-pointer-authentication-on-armv8-3.pdf)
- Roberto Avanzi - The QARMA Block Cipher Family ([Paper](https://eprint.iacr.org/2016/444.pdf) and [Presentation](https://www.nuee.nagoya-u.ac.jp/labs/tiwata/fse2017/slides/05-02.pdf))
- Roberto Avanzi - [Crypto that is Light to Accept](http://tce.webee.eedev.technion.ac.il/wp-content/uploads/sites/8/2016/05/light-crypto-public-2016.04.20.pdf)
- Rui Zong and Xiaoyang Dong - [Meet-in-the-Middle Attack on QARMA Block Cipher](https://eprint.iacr.org/2016/1160.pdf)

**Hardware**

- Ramtin Amin - [Lightning Connector](http://ramtin-amin.fr/#tristar)
- Ramtin Amin - [NVMe NAND Storage](http://ramtin-amin.fr/#nvmepcie)
- Ramtin Amin - [iPhone PCIe (dumping the 6s BootROM)](http://ramtin-amin.fr/#nvmedma)

## Write-Ups

- geohot - [evasi0n7](http://geohot.com/e7writeup.html)
- Jonathan Levin - TaiG 8.0 - 8.1.2 (Part [One](http://www.newosxbook.com/articles/TaiG.html) and [Two](http://www.newosxbook.com/articles/TaiG2.html))
- Jonathan Levin - TaiG 8.1.3 - 8.4 (Part [One](http://www.newosxbook.com/articles/28DaysLater.html) and [Two](http://www.newosxbook.com/articles/HIDeAndSeek.html))
- Jonathan Levin - [Who needs task_for_pid anyway?](http://newosxbook.com/articles/PST2.html)
- qwertyoruiopz - [About the “tpwn” Local Privilege Escalation](https://web.archive.org/web/20160131055957/http://blog.qwertyoruiop.com/?p=69)
- Ian Beer - [task_t considered harmful](https://googleprojectzero.blogspot.ch/2016/10/taskt-considered-harmful.html)
- jndok - [Exploiting Pegasus on OS X](https://jndok.github.io/2016/10/04/pegasus-writeup/)
- Siguza - [Exploiting Pegasus on iOS](https://siguza.github.io/cl0ver/)
- Ian Beer - mach_portal ([write-up](https://bugs.chromium.org/p/project-zero/issues/detail?id=965#c2) and [presentation slides](https://bugs.chromium.org/p/project-zero/issues/attachment?aid=280146))
- Ian Beer - [Exception-oriented exploitation on iOS](https://googleprojectzero.blogspot.ch/2017/04/exception-oriented-exploitation-on-ios.html)
- Jonathan Levin - [Phœnix](http://newosxbook.com/files/PhJB.pdf)
- Gal Beniamini - Over The Air (Parts [One](https://googleprojectzero.blogspot.ch/2017/09/over-air-vol-2-pt-1-exploiting-wi-fi.html), [Two](https://googleprojectzero.blogspot.ch/2017/10/over-air-vol-2-pt-2-exploiting-wi-fi.html) and [Three](https://googleprojectzero.blogspot.ch/2017/10/over-air-vol-2-pt-3-exploiting-wi-fi.html))
- Siguza - [v0rtex](https://siguza.github.io/v0rtex/)
- Ian Beer - [async_wake_ios](https://bugs.chromium.org/p/project-zero/issues/detail?id=1417#c3)
- Siguza - [IOHIDeous](https://siguza.github.io/IOHIDeous/)
- Jonathan Levin - QiLin ([PDF](http://newosxbook.com/QiLin/qilin.pdf) and [API](http://newosxbook.com/QiLin/))
- Brandon Azad - [A fun XNU infoleak](https://bazad.github.io/2018/03/a-fun-xnu-infoleak/)
- jeffball - [Heap overflow in necp_client_action](https://github.com/grimm-co/NotQuite0DayFriday/blob/master/2018.04.06-macos/notes.txt)
- xerub - [De Rebus Antiquis](https://xerub.github.io/ios/iboot/2018/05/10/de-rebus-antiquis.html)
- Ian Beer - [multi_path](https://bugs.chromium.org/p/project-zero/issues/detail?id=1558#c3)
- Brandon Azad - [blanket](https://github.com/bazad/blanket)
- Brandon Azad - [voucher_swap](https://googleprojectzero.blogspot.com/2019/01/voucherswap-exploiting-mig-reference.html)

## Other Lists

* qwertyoruiopz - iOS Reverse Engineering ([Wiki](https://github.com/kpwn/iOSRE/tree/master/wiki) and [Papers](https://github.com/kpwn/iOSRE/tree/master/resources/papers))
* Google Project Zero - [All the bugs Ian Beer has killed](https://bugs.chromium.org/p/project-zero/issues/list?can=1&q=reporter:ianbeer@google.com&sort=-closed&num=99999&colspec=ID%20Status%20Closed%20Reporter%20Methodology%20Summary)
