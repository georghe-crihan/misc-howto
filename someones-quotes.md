# The source code to Windows 3.1 KERNEL has been sitting under our noses since 2004
> Permanently Banned
> Posts: 158
> Joined: Thu Mar 02, 2017 6:40 am
> Post  by winnt32 » Sun Sep 06, 2020 8:21 am
>
> So I was reviewing the source leaks of Windows recently (782, 1381 sp1, and 2000 shell stuff) and was searching for super early NT related
> stuff such as i860 references when I hit upon a curious folder in the mdvm source code. One would assume that this would simply contain
> thunks - and indeed for most parts of Win16 it does - but for KERNEL it seems they just took the source code verbatim. There is even a log
> of changes going back several years included and comments going back to 1984! I haven't checked every WOW16 component yet, so some more might
> be found (most of it is just thunking code), but a critical part of Win16 source code jhas been sitting under our noses for about sixteen
> years now!

> The source code to MMSYSTEM, 16-bit Dr. Watson, 16-bit OLE1, Windows Write, parts of COMMDLG, TOOLHELP, the Win16 timing APIs, a few parts
> of USER (certain parts of the thunking code are simply directly copied from Win3.1 source), and some "SYSTEM" utility for determining system
> configuration (copyrighted 1983-1988) are also included.

> @Winins: mvdm is an early name for NTVDM - the DOS compatibility box for NT. Presumably it's based off of the OS/2 mdvm name.


# Re: The source code to Windows 3.1 KERNEL has been sitting under our noses since 2004
> Post  by jimmsta » Thu Sep 10, 2020 10:46 pm
>
> I ended up taking a look at the nt 3.5 version instead of the nt 4.0 version. It _is_ the 3.1 kernel, with WOW 1.0 ifdef's setup. It appears
> that it _can_ be compiled as a drop-in KERNEL.EXE for Windows 3.1, as all the makefiles exist (in the 3.5 version, anyway), as well as all
> includes. While it is a part of the MVDM/SoftPC module, it is provided as a complete copy of the 3.1 kernel with supportive wrappers if you
> attempt to compile it for NT. That's the key - you don't have to compile it for NT if you don't want to -- all the codepaths are provided for
> non-NT compilation. I'm not sure why the older 9yr old thread just dies with the second post -- perhaps the nt 4 iteration is changed heavily?
> I don't think I have that version readily available to look at, but I _do_ suspect that they did some serious modifications to it for NT4
> support (there's some hints in the comments that seem to indicate that they weren't done refining the MVDM module, and that they intended to
> do more work on it after the 1.0 (nt 3.1) release).
>
> There's also bits and pieces in the SoftPC bits that are leftover from the original XT-VDM project that SoftPC was derived from, going back
> to 1986. There's also the fake DOS 5 source code, implemented as part of the COMMAND.EXE dosbox interpreter, which is derived from the real
> DOS 5 code. There's a whole lot of neat stuff hidden in this that I don't think many people have taken a deeper look into.
>
> The SoftPC stuff on its own is a whole intricate beast - it was based on project that started in the mid-80's to implement an XT PC in software
> emulation, and was further developed to allow Macintosh's to run Windows and DOS in the late 80's. There's bits of that leftover in the headers,
> too. The whole thing is quite a powerful piece of software, and it was hidden within NT for the most part.
>

# Re: The source code to Windows 3.1 KERNEL has been sitting under our noses since 2004
> Post  by AltairMan » Sat Sep 12, 2020 8:46 am
>
> I never really looked into it before, but Win2K was a big change in how legacy 16-bit apps ran in the NT environment. NTVDM is supposed to be
> a 486 PC running DOS 5 (so maybe remnants of SoftPC?), and then WOW32 ran on top of that to provide the 16-bit environment.
>
> I pulled my copy of Windows Internals and see how parts of the kernal compared to what Pietrek wrote in the book. I only looked at one file, but
> it was interesting to follow the book along with the source.
>
> The other thing I'd like to see is VMM and the parts that make up WIN386. I know that sample code exists for some of the VxDs, but not all.

# Re: The source code to Windows 3.1 KERNEL has been sitting under our noses since 2004
> Post  by LOstDos » Wed Oct 07, 2020 6:49 am
>
> The book "SHOW STOPPER!" talks a little bit about it. The project (NT) needed a way to run all the popular programs of that era the NT was first
> developed, leading to emulation of DOS and Windows 16 bit for the non x86 platforms. Windows NT struggled a lot in the early days because of how
> slow it was compared to the real thing it was trying to do, because of early design goals and rules.
>
> And yes, it is true that the kernel is all in there. The main proof of course being Matt Pietrek's Windows Internals book going into details how
> the memory and task scheduling works.

# Re: The source code to Windows 3.1 KERNEL has been sitting under our noses since 2004
> Post  by AltairMan » Fri Jan 01, 2021 10:56 am
>
> Not sure why the thread pointed to by @voidp dies either. I don't know if anyone on this thread tried building what's there; I was going to play
> with it today. There's a thread elsewhere on BA (viewtopic.php?t=6334) that kind of describes setting the build environment properly. I'm
> assuming that installing VS6 yields build tools. But, looking at the distribution, there is a pretty complete set of tools in
> NT4\nt\private\windows\media\avi\bin.16. It's missing one or two random things, though. There are a lot of environment scripts that have network
> names and mappings.
>
> Right now I'm trying to compile one or two of the simplest SDK tools that's not in the actual SDK distribution and it keeps barfing because of
> how environment variables are used. Makes total sense for a production build environment.

