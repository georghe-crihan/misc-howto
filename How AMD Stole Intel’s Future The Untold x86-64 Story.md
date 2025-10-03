Transcript from [Dave Plummer's](https://www.linkedin.com/in/davidplummer) Youtube channel:
https://www.youtube.com/watch?v=WyX8TO3awfw

# Chip Wars: Why Itanium Died and x64 Took Over | Inside the Windows Port

Hey, I'm Dave. Welcome to my shop. I'm Dave Plamer, a retired software engineer
from Microsoft going back to the MS DOS and Windows 95 days.

And today, we're going to answer a deceptively simple question, one that has shaped
20 years of desktop and server computing. How did AMD managed to leapfrog Intel using
Intel's own instruction set? It's the X64 story, but with a plot twist.

Because for a decade, everyone's hopes were pinned on Itanium, the gleaming
*Very Long Instruction Word*[^VLIW] feature.
[^VLIW]: *VLIW* - Very Long Instruction Word is a CPU technology where an
instruction contains multiple operations that can be executed in parallel by
different functional units..

And just about the time that the orchestra
swelled for its triumphant entrance, the Itanium missed its queue. AMD, the
scrappy underdog from Sunnyvale, was already in the wings with X86-64. And
when the baton slipped, they caught it in stride and never looked back. If you
were on the Windows side when Dave Cutler's team brought x64 into NT, you
saw the whole thing unfold from the kernel on outwards. If you glanced at
Task Manager on those early x64 builds, you might have noticed a little tiny
asterisk that told a big story. But first, let's rewind to set the stage.

By the late 1990s, 32-bit x86 had become a hugely successful accident of history.
It started life in the late '70s as the 8086, hardly designed for the gigantic
threaded graphical multitasking worlds we'd eventually build. But x86 was
everywhere, cheap and compatible with the mountains of software that the world
already relied on. That compatibility was both its superpower and its shackle.
A lot of people, some very smart ones, concluded that we'd eventually need to
break out of the backward compatibility prison and start fresh. So, Intel and HP
did exactly that with Itanium, betting on *EPIC*[^EPIC], *Explicitly Paralleli
Instruction Computing*. In *EPIC*, the compiler took on the burden of scheduling
the instruction
stream statically, bundling operations and predicting resource hazards ahead of
time. In theory, it would unlock incredible parallelism by extracting the
last bits of instruction level parallelism from your code. In practice,
the compilers had to be near omniscient.  The binary ecosystem had to be rebuilt,
and real world performance wound up coming in somewhere between
underwhelming and mortifying, especially if you're running existing x86 code
under emulation.

Now, the thing about revolutions is you have to bring the entire city along. The
plumbers, the bakers, the street sweepers, all the shops, and all the signage.
Intel's revolution swapped out the language of the city, the alphabet on the street
signs, and then asked every store owner to repaint their awnings by Tuesday. And
while the few storefronts that did switch sometimes found Itanium ran like
a scolded cat on certain HPC kernels that the compilers could schedule
cleanly, the rest of the city mostly stayed put. Microsoft produced an
Itanium build of Windows, SQL Server, the whole bit. But the gravitational
pull of compatibility remained enormous because the real world is messy.
Drivers, legacy apps, software that was never really portable, and entire
business processes welded to a specific ABI [^ABI] that all refused to move as one.
[^ABI]: *ABI* - Application Binary Interface - the mechanism by which programs
call into the operating system or other libraries.

And
that's exactly where AMD saw the seam in its universe. Instead of overthrowing
x86, they embraced and extended it. The design goal for their x86-64[^x86-64], which
AMD later branded as AMD64, was quietly radical.
[^x86-64]: *x86-64* - AMD's proposed instruction set that extended the x86 set while
retaining backward compatibility with it.

Keep the programming model
familiar enough that OS vendors and compilers could come along with moderate
work, yet fix the architectural sins that throttled 32-bit performance and
addressing. You could run 32-bit code essentially unmodified on a 64-bit
operating system because the hardware provided compatibility mode under the
umbrella of Long Mode[^LongMode].
[^LongMode]: *Long Mode* - Supports larger memory (up to 2^64 bytes), enhanced
performance, and compatibility with 32-bit applications via Compatibility Mode.

User mode 32-bit apps could use their same old
instruction set and call into the 64-bit kernel through a thin WOW64[^WOW64] veneer.
[^WOW64]: *WOW64* - Microsoft's solution for running 32-bit Windows applications
"natively" on 64-bit Windows.

Meanwhile, for native 64-bit processes, you got a cleaned up programming model
with the rough edges of 1980's segmentation[^Segmentation] quietly swept away.
[^Segmentation]: - *Segmentation* The method of reaching higher addresses with a CPU
by combining a segment and offset register to yield a final address. It's not pretty
and it's what the x86 used.

In long
mode, segmentation is effectively gone. *FS* and *GS* remain as base registers for
thread local storage, but the rest of the old segment arithmetic that made
compiler writers reach for their Aspirin was finally retired. That alone
simplified a mountain of code generation and runtime complexity. AMD also added
what the 32-bit x86 really starved for, more general purpose registers. I mean,
I grew up on 6502, which has one accumulator and two 8-bit index
registers. So 8 always felt like a lot to me, but compiler authors did not
agree. The original 8[^Original8], which are *EAX* through *EDI* with *ESP*
and *EBP* as stack and frame pointers, were never enough for the modern compilers.
[^Original8]: *Original 8* - The original eight 32 bit registers: EAX, EBX, ECX, EDX,
ESI, EDI, EBP, ESP.

Every time the
register allocator ran out of registers, it had to spill variables onto the
stack. And that's death by a thousand cache misses. AMD-64 gives you 16 full
64-bit general purpose registers *RAX* through *RA15*, plus 16 XMM registers
baseline by Fiat, because SSSE2[^SSE2] is mandatory in 64-bit mode.
[^SSE2]: *SSE2* - Adds support for 128-bit SIMD (Single Instruction, Multiple Data)
operations on double precision floating point numbers and integers. Smoking fast when
used. 

The extra registers
turned out to be profoundly important. A lot of code gets faster 64-bit, not
because it suddenly needs 64-bit pointers but because the compiler finally has
register breathing room and a saner calling convention. And about those pointers,
AMD's designers were conservative in the best sense. Rather than flipping the entire
virtual address range from 32 bits to a full 64, they defined canonical addresses and
initially only used the low 48 bits of virtual addresses, sign extended through
the upper bits. That made page table structures tractable. Four levels of
paging with *PML4*[^PML4] at the top and left room for future expansion.
[^PML4]: *PML4* - The top-level structure in the four-level paging hierarchy used in
x86-64 (AMD64) Long Mode for virtual-to-physical address translation.

Physical
address width also grew modestly at first. The idea was that OS vendors
shouldn't have to re-engineer every allocator and metadata structure in the
kernel to handle 64-bits everywhere overnight. Instead, they could move
deliberately, picking up the right 64-bit types for the paths that needed
them and keeping structures tight elsewhere.

Two other surgical fixes speak to the elegance of the design. First, AMD introduced
*RIP-relative*[^RIPRelative] addressing, because on 32-bit x86, position-independent
code was a contortion act.
[^RIPRelative]: *RIP-Relative* - Code and data implicitly based of the address of the
*RIP* register, so that it can be executed at an arbitrary location in memory more
easily.

You wound up loading addresses into registers and indexing
through them. With *RIP* relative addressing, code can refer to nearby
data using offsets from the instruction pointer. That simplified loaders made
shared libraries cleaner and reduced fixups. Intel had introduced *SYSENTER*[^INT2E]
and *SYSEXIT* initially on the 32-bit chips and on x64.
[^INT2E]: *INT 2E* - the instruction used for a kernel transition on older X86 code.
Replaced by *SYSCALL* and *SYSRETURN* on AMD64.

AMD's *SYSCALL* pair
became the clean fast path and you get it every time that you call into the OS.

Meanwhile, back on the farm, let's check in on Microsoft. Inside NT, moving the
kernel to a new architecture is never trivial, but AMD had made it feel like
upgrading a house rather than pouring a new foundation from scratch. The memory
management layer had to learn 64-bit page tables, but the abstractions were
still familiar. The trap handlers were rewritten for the new calling
conventions and the syscall path, but the kernel's responsibilities (scheduling, I/O,
memory management) didn't suddenly change under the new system. *WOW64* could host
32-bit userland in a set of 32-bit NTDLL and system DLLs living beside the 64-bit
world. Thunking[^Thunking] was narrow and mechanical, not a crazy research project.
[^Thunking]: *Thunking* - Jumping from 32-bit to 64-bit, or copying data betwee them
(or vice versa). Also done between 16 and 32-bit Windows.

Drivers were the sore point though, because you can't load a 32-bit driver into a
64-bit kernel any more than you can bolt a lawnmower carburetor onto a jet engine.
So, the device ecosystem had to be rebuilt. But the NTDDK[^NTDDK] did what it's
always done.
[^NTDDK]: *NTDDK* - Windows NT Device Driver Kit, the software that you use to help
you write device drivers for Windows.

It put a stable facade over the guts. So most authors were able to just port or
recompile rather than rewrite. Because Task Manager had always done its internal
accounting using full 64-bit types, never assuming, for example, that PIDs fit in
this many bits or that memory counters would forever be 32-bits. It just worked when
recompiled for x64. The only visible change was cosmetic. Dave C. added an asterisk
to the process column to mark 64-bit processes. And under the hood, it actually got
a lot faster. Not because a process list suddenly needs 16 exabytes[^Exabyte] of
address space, but because the 64-bit calling convention passes arguments and
registers, and the compiler has room to keep hot variables[^HotVariables] out of
the stack.
[^Exabyte]: *Exabyte* - The amount of system memory consumed by a Chrome Tab when
loading. 

[^HotVariables]: *"HOT" Variables* - things that are used or referenced all the time,
so making access to them faster speeds up the entire system as a result.

The best porting stories are like that. The light comes on and the counters spin
just the same as always, but quicker.

Let's talk calling conventions a bit because this is where AMD's keep it
familiar but better philosophy really paid off. On Win32, you live the world
of standard call and fast call flavors and almost everything passed on the
stack. On Win64, Windows standardized a single convention. The first four
integer or pointer parameters go in *RCX*, *RDX*, *R8*, and *R9*. Floating point
arguments track in *XMM0* through *3*, and the caller reserves a 32 byte shadow
space on the stack for the collee to spill there if they need it. Callee
saved registers are *RBX*, *RBP*, *RSI*, *RDI*, and *R12* through *15*, which gavei
the compiler a broader register file and predictable unwind semantics. Structured
exception handling switched to table-based unwinds with precise metadata. And because
x64 mandated SSE2, nobody had to juggle the ancient x87 stack for floating point,
except if you were interacting with old code. You can trace significant chunks of
the performance boost in boring server workloads directly to those types of
decisions.

And while we're in the weeds a little bit here, let's take a look at the memory
protection story. AMD wired in the *NX* bit[^NXBit] or the *No eXecute* bit as a
firstclass citizen in the page tables.
[^NXBit]: *"NX" Bit* - Marks a page of memory as data only so that malware can't
poke itself into memory, like on the stack, and then run itself.

Intel later adopted it as *XD*. Windows
turned that into *DEP*[^DEP], *Data Execution Prevention*, long before "Write With
Execute Disabled" became cocktail chatter.
[^DEP]: *DEP* - Data Execution Prevention - The Windows feature that uses the NX bit
to protect important memory from malware exploits.

AMD64 plus *DEP* plus *ASLR* moved
the needle on exploit resistance in a very real way. Some of the most impactful
security improvements of the mid-2000s were unlocked by the new ISA's page table
semantics as much as by any special kernel hardening.

If you want to see how architecture becomes strategy, compare the AMD64's approach
with Itanium's. With *IA64*[^IA64], the compiler is king.
[^IA64]: *IA64* - Used in the Itanium family of processors. Unlike x86-64 (AMD64),
IA-64 is a distinct architecture based on Explicitly Parallel Instruction Computing
(EPIC).

You recompile everything for the
new world and let the optimizer discover the parallelism explicitly. With AMD64,
the micro architecture is the plumber. You leave most of the instruction set
architecture semantics alone and make the pipes bigger and straighter. The
really cool twist is that the micro-architectural part of AMD's bet paid off
twice. First, the K8 Hammer cores that shipped in Opteron and Athlon64 were
just flatout good CPUs. And they had an integrated memory controller, so you
didn't pay the penalty of a front side bus hop to fetch every cache line. They
used *HyperTransport*[^HyperTransport], rather than a shared *Front Side Bus*, so
multi-socket Opteron and scaled pretty elegantly.
[^HyperTransport]: AMD HyperTransport is a high-speed, low-latency point-to-point
link for interconnecting integrated circuits, such as btween multiple CPUs on a
motherboard.

And the pipeline wasn't just the heat soaked
spaghetti factory that Intel's Prescott Netburst eventually became. When Intel
aimed for 10GHz by stretching the pipeline and shrinking the useful work in each
cycle, they drifted into a thermal cul-de-sac. AMD stayed tight and fast. So when
Windows Server and SQL Server compiled clean for the AMD64, the platform delivered
honest to goodness throughput. Second, the AMD64 instruction set itself gave
compilers the elbow room that they'd always wanted. Recompile your middleware as
x64, and you saw gains even if your data structures didn't grow simply because
spilling[^Spilling] went down, inlining got smarter, and the register allocator
stopped living in constant debt.
[^Spilling]: *Spilling* - When the compiler has to use stack space rather than fast
registers to hold a variable that is being used by a section of code. More registers
means less spilling!

Meanwhile, the OS team at Microsoft could push out a 64-bit Windows that didn't
strand the application ecosystem. WOW64 is a neat example of AMD's design enabling
a clean system. In long mode, the CPU executes 32-bit user code in compatibility
mode, while the kernel stays 64-bit. The transitions through NTDLL and the system
call stubs take on the necessary gymnastics, so that from the app's perspective,
it's just making the same win32 calls that it always did, and the kernel never
stops being a 64-bit citizen. That's also why you wind up with SYSWOW64 containing
32-bit binaries and system32 containing 64-bit binaries. The naming stayed backward
looking for compatibility reasons because that's where people looked for those files.
The contents flipped though.

File system register redirection kept old installers from scribbling on the wrong
hives. The price you paid was that 32-bit drivers were persona non grata and 16-bit
apps finally saw the sun go down. See you later, Machinist's Friend. Pour one on
the curb. But the smoothness of the port brought Microsoft enough political capital
to enforce two big quality levers on x64: driver signing and patch guard, which is
kernel patch protection. Those weren't just incidental. The platform shift was
strong enough that Redmond could come along and say, "Hey, new world, new rules."
From a developer's chair, moving code from 32-bits to 64-bits kind of fell into two
buckets. If you were living in userland and your code was well behaved, I mean,
you used *size_t* when you meant pointer size quantity and stuff and *DWORD* for
32-bit integers when you meant 32-bit integers and you didn't smuggle pointers
through your *LONG*s and stuff, then your worst days were about format strings and
structure padding with *printf* needing *%p* instead of *%08x* for pointers and
stuff like that. You found out that *time_t* or *size_t* had grown if you look
behind the scenes and your custom memory allocator needs to think
in terms of real size of pointers and alignment, if you were into that. The
compilers helped a lot because warnings about truncation and sign extension went
from "That's cute!" in the old days to "Fix it or this won't link!". A sloppy
codebase that had muddled the distinction between integer width and pointer width
learned humility quickly. Kernel mode was its own little amphitheater. *HRPs*, *MDLs*,
and the memory manager data structures picked up the 64-bit pointers. DMA code
had to understand devices that could only address 32 bits of physical space.
I/Octal packing had to marshall 32-bit user buffers correctly. But none of it
required new mathematics, just discipline.

There were also a few pleasant surprises because the x64 calling convention moved
the first four parameters into registers. Certain hot abstractions got noticeably
cheaper. Function pointer callbacks, *COM v-tables*, and even your basic C++ object
calls saw their overhead shrink. *RIP* relative addressing made position independent
code and therefore DLL code cleaner. And with the SSE2 baseline, your vector math
could finally assume a modern floatingoint substrate. That's partly why even GUI
infrastructure like GDI and USER32 felt sprite layer in native 64-bit builds. Even
things like Task Manager got a little faster because when you remove unnecessary
spills and stack traffic, even aggressively optimized utilities get out of their
own way.

Meanwhile, out in the real world of the market, the baton handoff was happening
in public. In 2003, AMD shipped Opteron and then Athlon64. Microsoft shipped
Windows Server on AMD64 and not much later, Windows XP Professional x64 Edition.
ISV started doing the math and decided they could ship a 64-bit build and keep their
32-bit binary for customers still on the old hardware. And then came the moment that
cemented the plot twist. Intel adopted AMD 64-bit extensions. They tried to call it
EM64T for a while and then Intel 64, but the die was cast. It was x86-64. That was the
new *Lingua Franca*. And when your rivals adopt your instruction set architecture,
you haven't just won a technical argument. You've redrawn the map. Now, people on
lesser channels would simplify this as AMD beat Intel because Intel bet on Itanium
and Itanium Dumb. But that leaves out half the truth. AMD didn't just get lucky that
Itanium stumbled. They engineered an extension to x86 that was so compatible, so
practical, and so strategically timed that the OS and compiler vendors could make
it real without burning down the world. Yes, Itanium's compiler-driven *EPIC*
vision proved brittle, but AMD64 success depended on a thousand careful paper cuts.
[^EPIC]: *EPIC* - Explicitly Parallel Instruction Computing. Requires the compiler to
detect and leverage parallelism in the code.

Mandate
SSE2, so the floating point story is sane. Add registers, so compilers stop thrashing.
Remove segmentation to simplify code generation. Preserve a flat model supporting is
viable. Wire in the *No eXecute* and *DEP*'s bits so Ops teams can sell security and
bake a system call path in that lets the kernel breathe.

K8's micro-architecture then turned all of that potential into performance at a
time when Netburst was just melting clocks for breakfast. One of Dave
Cutler's imprints on Windows has always been about ruthlessly clear abstractions
and discipline at the edges. They don't lie about sizes. They don't pretend a
32-bit quantity and an address are the same thing. They don't let calling
conventions metastasize. And they try to keep the kernel small, testable, and
hostile to undefined behavior. Those principles turned out to be the perfect
attitude for a 64-bit transition.

Windows type system *size_t* *ulong* pointer and the split of the integer widths
wasn't just pedentry. It was laying the land markers for this exact day. That's
why when someone like you reports that Task Manager just worked and they picked
up an asterisk to be native 64 bits, I smile because that's what careful engineering
looks like from the outside. Sometimes it doesn't look like anything
at all because it just works.

There's another little piece of the story that's kind of fun in retrospect. For
years on 32-bit Windows, the address space was basically split: 2 gigabytes went to
the user space and 2 gigabytes went to the kernel and later you could change that
to three gigabytes for certain server apps and it was a constant negotiation. *Large
Address Aware* flags, *AWE*, *PAE* and a thousand creative schemes were used to
stuff ever larger data sets into a 32-bit virtual address space that was
never designed for them. On x64 the grunt work falls away. User mode starts
in a virtual address universe so enormous that the constraints shift from
"How do I map it?" to "How do I keep my caches happy with it?" Kernel pools get
elbow room and file system caches stop worrying about starving other callers.
Memory mapped I/O for modern PCIe devices has room to breathe. The sense of relief
was palpable among server developers. The thing you fought all day just became
a non-problem is how you often looked at it. You can draw a straight line from
AMD catching that baton to the platform norms that we now take for granted. We
moved to a security posture of *No eXecute*, a meaningful *ASLR* because the
hardware gave the OS clean primitives.  We standardized on a single modern
calling convention and we mandated a floating point substrate that every
compiler could count on. We saw operating system vendors and not just
Microsoft enforce stronger driver quality gates because coming along with
the disruption of an entirely new architecture, they finally could. They
had the chance. And when Intel eventually abandoned Netburst, embraced
Core, and then rolled in their own excellent micro-architectures, the war
returned to where it belongs - cores, caches, fabrics, and efficiency. On the
same shared ISA that AMD had made inevitable. If you want a cautionary
epilogue, think about all the designs that never quite ship because they're
technically pure, but operationally impossible. Itanium wasn't wrong so much
as it was incompatible with the way real software is built and deployed. AMD64
wasn't perfect. Nothing is, but it respects the entanglements of the world
as it is. The payoff for that humility was that Microsoft could bring forward
NT without a score rewrite and your tools team could switch Visual C++ to
target x64 without rewriting half the optimizer, and your old friend Task
Manager could just flip the compiler switch, pick up a star, and go home
early that day.

So, how did AMD leapfrog Intel using Intel's own instruction set?  By refusing to
fight the last war, they quietly rebuilt the road while everybody else was planning
a new city. They added width where it mattered, removed historical friction and kept
the contract with all the software that already existed. When Itanium stumbled,
AMD64 didn't have to run. It was already there, tested, toolchained, and delivering
throughput. Intel's eventual adoption of x86-64 sealed the deal. But the leapfrog
had already happened. The moral for engineers is the same one Cutler taught by
example. Don't confuse elegance with utility. Design in ways that let reality say
yes, and you might just find the market handing you the baton when it counts.

Thanks for joining me out here in the shop today. Make sure you're subscribed. I'll
see you next time.
