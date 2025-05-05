Q: How do I debug DOS4GW programs without source code?
A: Create S-ICE_PM directory
- Take the WIN386.EXE from the Windows 3.1 distribution, rename it to Win.Com, move to S-ICE_PM.
- Take the COMMAND.COM from the DOS distribution, rename it to KRNL386.EXE, move to S-ICE_PM.
- Take the following files from WIN-ICE distribution: WINICE.EXE, WINICE.DAT, WLDR.EXE, move to S-ICE_PM.
- Optionally you could also add SERIAL.EXE, UPTIME.EXE, WLOG.EXE, but avoid adding WINICE.VID.
- In the S-ICE_PM directory, create the SYSTEM.INI file with the following content:

```
[386Enh]
mouse=*vmd
network=*dosnet,*vnetbios
ebios=*ebios
display=*vddvga
keyboard=*vkd
device=*vpicd
device=*vtd
device=*reboot
device=*vdmad
device=*vsd
device=*v86mmgr
device=*pageswap
device=*dosmgr
device=*vmpoll
device=*wshell
device=*BLOCKDEV
device=*PAGEFILE
device=*vfd
device=*parity
device=*biosxlat
device=*vcd
device=*vmcpd
device=*combuff
device=*cdpscsi
local=CON
FileSysChange=off
PagingFile=C:\WIN386.SWP ; To be changed to your liking and requirements :)
MaxPagingFileSize=45056
MinTimeslice=20
WinTimeslice=100,50
WinExclusive=0
```

To suppress the "invalid switch" message, patch the WINICE.EXE file: replace the sequence 20 2F 33 0D with 0D 00 00 00.
Launch WINICE.EXE and enjoy.
If you already have a Windows installation, but you'd like to add the S-ICE_PM to your Path environment variable,
it won't hurt to rename the win.com to, say wnk.com and patch the Winice.exe to replace the "win." with "wnk.".

# NOTE: source availability
- MS-DOS 4.0 opensourced by Microsoft. (Minimal MS-DOS version required to run Windows 3.1 in 386 enhanced mode is 3.1).
or you could use DOSBOX for which the source is available.
FreeDOS can be used, although the kernel has to be compiled with `WIN31SUPPORT` define enabled and Windows 3.11 `system.ini` containing
`[386Enh]` with the flag `InDOSPolling=TRUE` enabled and DOS file sharing (`share.com`) loaded. That is `/DWIN31SUPPORT` compile
time switch.
- WIN386.EXE (the Windows 3.1 DPMI host) could be available elsewhere, as part of the Windows 3.x leaked sources.
Look for the `win386.mk` file.
