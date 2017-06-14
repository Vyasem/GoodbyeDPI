GoodbyeDPI — Passive Deep Packet Inspection blocker and Active DPI circumvention utility
=========================

This software designed to bypass Deep Packet Inspection systems found in many Internet Service Providers which block access to certain websites.

It handles DPI connected using optical splitter or port mirroring (**Passive DPI**) which do not block any data but just replying faster than requested destination, and **Active DPI** connected in sequence.

**Windows 7, 8, 8.1 and 10** with administrator privileges required.

# How to use

Download [latest version from Releases page](https://github.com/ValdikSS/GoodbyeDPI/releases) and run.

```
Usage: goodbyedpi.exe [OPTION...]
 -p          block passive DPI
 -r          replace Host with hoSt
 -s          remove space between host header and its value
 -f [value]  set HTTP fragmentation to value
 -e [value]  set HTTPS fragmentation to value
 -a          additional space between Method and Request-URI (enables -s, may break sites)

 -1          -p -r -s -f 2 -e 2 (most compatible mode, default)
 -2          -p -r -s -f 2 -e 40 (better speed yet still compatible)
 -3          -p -r -s -e 40 (even better speed)
 -4          -p -r -s (best speed)
```

Try to run `goodbyedpi.exe -1 -a` first. If you can open blocked websites it means your ISP has DPI which can be circumvented. This is the slowest and prone to break websites mode, but suitable for most DPI.

Try `-1` to see if it works too.

Then try `goodbyedpi.exe -2`. It should be faster for HTTPS sites. Mode `-3` speed ups HTTP websites.

Use `goodbyedpi.exe -4` if it works for your ISP's DPI. This is the fastest mode but not compatible with every DPI.

# How does it work

### Passive DPI

Most Passive DPI send HTTP 302 Redirect if you try to access blocked website over HTTP and TCP Reset in case of HTTPS, faster than destination website. Packets sent by DPI have always have IP Identification field equal to `0x0000` or `0x0001`, as seen with Russian providers. These packets, if they redirect you to another website (censorship page), are blocked by GoodbyeDPI.

### Active DPI

Active DPI is more tricky to fool. Currently the software uses 4 methods to circumvent Active DPI:

* TCP-level fragmentation for first data packet
* Replacing `Host` header with `hoSt`
* Removing space between header name and value in `Host` header
* Adding additional space between HTTP Method (GET, POST etc) and URI

These methods should not break any website as are fully compatible with TCP and HTTP standards, yet it's sufficient to prevent DPI data classification and to circumvent censorship. Additional space may break some websites, although it's acceptable by HTTP/1.1 specification (see 19.3 Tolerant Applications).

The program loads WinDivert driver which uses Windows Filtering Platform to set filters and redirect packets to the userspace. It's running as long as console window is visible and terminates when you close the window.

# How to build from source

This project can be build using **GNU Make** and [**mingw**](https://mingw-w64.org). The only dependency is [WinDivert](https://github.com/basil00/Divert).

To build x86 exe run:

`make CPREFIX=i686-w64-mingw32 WINDIVERTHEADERS=/path/to/windivert/include WINDIVERTLIBS=/path/to/windivert/x86`

And for x86_64:

`make CPREFIX=x86_64-w64-mingw32 WINDIVERTHEADERS=/path/to/windivert/include WINDIVERTLIBS=/path/to/windivert/amd64`

# How to install as Windows Service

One way is using an [srvstart](http://www.rozanski.org.uk/software) program.  
Unpack it to goodbyedpi directory and create 3 files:

*goodbyedpi.ini*
```INI
[GoodByeDPI]
startup=goodbyedpi.exe
shutdown_method=winmessage
auto_restart=n
```
*srvinstall.bat*
```Batchfile
srvstart install GoodByeDPI -c %CD%\goodbyedpi.ini
```
*srvremove.bat*
```Batchfile
srvstart remove GoodByeDPI
```
Run these batch files as Administrator to install/remove service.  
Open Windows Services panel to run service or make it start automaticaly.

# Similar projects

[zapret](https://github.com/bol-van/zapret) by @bol-van (for Linux).

# Kudos

Thanks @basil00 for [WinDivert](https://github.com/basil00/Divert). That's the main part of this program.

Thanks for every [BlockCheck](https://github.com/ValdikSS/blockcheck) contributor. It would be impossible to understand DPI behaviour without this utility.