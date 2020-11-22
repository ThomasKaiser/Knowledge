# Exploring Apple Silicon on a MacBookAir10

**WORK IN PROGRESS**

## Basics

As of Nov 2020 Apple startet to produce Mac computers based on their own SoCs instead of relying on Intel/AMD CPUs/GPUs. Below the M1 SoC with two LPDDRX4 modules soldered right next to it on a Mac Mini mainboard ([image source](https://egpu.io/forums/desktop-computing/teardown-late-2020-mac-mini-apple-silicon-m1-thunderbolt-4-usb4-pcie-4/)):

![](../media/m1-with-lpddr4x-ram.jpg)

While most people talk about the ISA only (switching from Intel to ARM) the transition is a bit more than this since those Apple SoCs also contain a lot more than just CPU cores (GPU and Machine Learning cores, SSD and memory controller, various accelerators for video, crypto, etc.)

For a more broad overview what to expect with the new platform check [Anandtech](https://www.anandtech.com/show/16252/mac-mini-apple-m1-tested) or others. I try to focus on the stuff that is not mentioned everywhere else on the Internet.

## Gathering information

### The lazy way

Setting this new laptop up the easy/lazy/noob way: finishing last TimeMachine backup on my i7 MacBook Pro 16", firing up the new MacBook Air, connecting the TM disk to the Air and let Migration Assistant transfer everything (this includes a lot of stuff below `/usr/local/` e.g. my whole [homebrew](https://brew.sh) install and some other `x86_64` binaries).

After this migration, a macOS update to 11.0.1 and a reboot everything works as it should. Safari starts with the ~60 tabs open I had on the Intel MBP, my text editor opens with lots of unsaved documents but random `x86_64` CLI tools below `/usr/local/bin/` do not execute. I needed to fire up one of the many Intel applications below `/Applications/` so I've been asked whether I want to install Rosetta2. After this step all my `x86_64` stuff works since Rosetta2 kicks in with first invocation and does the 'binary translation' job so from then on the translated binary will be invoked directly.

I realized one minor exception when I wanted to explore binaries with the `lipo` tool since all the 'Xcode command line tools' remained in `x86_64` and there was a library mismatch. Reinstalling the Universal Binary version of the tools did the trick:

    sudo rm -rf /Library/Developer/CommandLineTools/
    sudo xcode-select --install

For now I'm staying with the whole userland below `/usr/local/` being `x86_64` so everything I run from there has been translated using Rosetta2 to `arm64e` once. This is mostly since homebrew is yet not ready for ARM (some formulas will break) so it's important to stay on the 'wrong' architecture when dealing with homebrew. To install for example a tool to examine/use transparent filesystem compression I can't call `brew` directly but have to use the `arch` command instead:

    arch -x86_64 brew install afsctool

(an alternative when working directly sitting in front of the machine is to tick the 'open in Rosetta' checkbox of Terminal.app or iTerm.app in Finder since the architecture the tool is running with will be inherited to every process executed from within)

### First impressions

The "Apple Silicon" MacBook Air feels faster than my 16" Intel MacBook ([Core i7-9750H](https://ark.intel.com/content/www/de/de/ark/products/191045/intel-core-i7-9750h-processor-12m-cache-up-to-4-50-ghz.html), also 16 GB RAM), opening applications with lots of documents (~60 tabs in Safari, ~50 partially large text documents in BBEdit) is faster, sleep/wake is literally immediate but sound output is not that great as on the larger Intel MacBook ;)

### Basic collection of information

  * `ioregl -l` output: https://github.com/ThomasKaiser/Knowledge/tree/master/media/ioreg-MacBookAir10.txt
  * `hidutil list` output: http://ix.io/2EUV (19 thermal sensors inside)
  * `system_profiler` output: http://ix.io/2EVx
  * When looking at CPU utilization (be it with `htop`, `Activity Monitor` or `powermetrics`) it's important to realize that cpu0-cpu3 are efficiency cores and cpu4-cpu7 are the performance cores.

### CPU performance assessment

Since being an energy efficiency fetishist I play around with ARM single board computers as miniature servers for a while. To benchmark those things I found 7-zip's internal benchmark a pretty good representation of ['server workloads in general' since relying on integer/memory](https://github.com/ThomasKaiser/sbc-bench#7-zip). Of course it makes absolutely no sense at all to use this benchmark on a laptop due to totally different use cases.

But I gave it a try just to compare with my Intel MacBook. `/usr/local/bin/7z b` ends up with ~27500 7-zip MIPS (running in Rosetta2 'emulation' mode!). After half an hour some slight throttling kicks in and the results are in the 26000-26500 range. The MacBook Air has no fan, is passively cooled and doesn't even get hot (touch test). The very same benchmark running natively on my Intel i7-9750H MacBook Pro (6 cores, 12 threads) results in 29000 7-zip MIPS while the fan kicks in after a minute and screams at +5000 rpm and a little later the CPU thermal sensor reports temperatures above 90°C with fan on maximum operation.

I won't do any further CPU performance testing since 'fast enough' or let's better say *simply impressive* considering the thermal behaviour and that I tested an Intel binary on the ARM laptop running via binary translation. I'll focus on performance/watt measurements solely later on.

### Internal storage

Fortunately the SSD is still exposed as NVMe storage so SMART queries will work. `ioreg -l` shows

    "IOClass" = "AppleANS3NVMeController"
    "IOPolledInterface" = "IONVMeControllerPolledAdapter is not serializable"
    "IOMaximumSegmentByteCountRead" = 4096
    "IOPlatformPanicAction" = 0
    "IOPersonalityPublisher" = "com.apple.iokit.IONVMeFamily"
    "IOReportLegendPublic" = Yes
    "IOCommandPoolSize" = 64
    "IOProviderClass" = "RTBuddyService"
    "Physical Interconnect Location" = "Internal"
    "IOMaximumSegmentByteCountWrite" = 4096
    "IOMaximumSegmentCountRead" = 256
    "Model Number" = "APPLE SSD AP0512Q"
    "IOProbeScore" = 300000
    "IOPowerManagement" = {"DevicePowerState"=1,"CurrentPowerState"=1,"CapabilityFlags"=32768,"MaxPowerState"=1}
    "IOMaximumSegmentCountWrite" = 256
    "Serial Number" = "---"
    "NVMe Revision Supported" = "1.10"
    "IOPropertyMatch" = {"role"="ANS2"}
    "AppleNANDStatus" = "Ready"
    "Chipset Name" = "SSD Controller"
    "Physical Interconnect" = "Apple Fabric"
    "CFBundleIdentifierKernel" = "com.apple.iokit.IONVMeFamily"
    "Vendor Name" = "Apple"
    "CFBundleIdentifier" = "com.apple.iokit.IONVMeFamily"
    "IOMatchCategory" = "IODefaultMatchCategory"
    "IOMaximumByteCountRead" = 1048576
    "IOMaximumByteCountWrite" = 1048576
    "IOMinimumSegmentAlignmentByteCount" = 4096
    "Controller Characteristics" = {"default-bits-per-cell"=3,"firmware-version"="1161.40.","controller-unique-id"="0ba0102ae3acd80d    ","capacity"=512000000000,"pages-per-block-mlc"=1152,"pages-in-read-verify"=384,"sec-per-full-band-slc"=52224,"pages-per-block0"=0,"cell-type"=3,"bytes-per-sec-meta"=16,"Preferred IO Size"=1048576,"program-scheme"=0,"bus-to-msp"=(0,0,1,1),"num-dip"=34,"nand-marketing-name"="itlc_3d_g4_2p_256               ","package_blocks_at_EOL"=31110,"sec-per-full-band"=156672,"cau-per-die"=2,"page-size"=16384,"pages-per-block-slc"=384,"sec-per-page"=4,"nand-device-desc"=3248925,"num-bus"=4,"block-pairing-scheme"=0,"chip-id"="S5E","Encryption Type"="AES-XTS","vendor-name"="Toshiba         ","blocks-per-cau"=974,"dies-per-bus"=(5,4,4,4),"msp-version"="2.8.7.0.0       ","manufacturer-id"=<983e99b3fae30000>}
    "Firmware Revision" = "1161.40."
    "IOReportLegend" = ({"IOReportChannels"=((5644784279684675442,8590065666,"NVMe Power States")),"IOReportGroupName"="NVMe","IOReportChannelInfo"={"IOReportChannelUnit"=72058115876454424}})
    "DeviceOpenedByEventSystem" = Yes

See "Controller Characteristics" for some internals. `ioreg` entry for the `IOEmbeddedNVMeBlockDevice`:

    "Logical Block Size" = 4096
    "IOMaximumBlockCountWrite" = 256
    "IOMaximumSegmentByteCountRead" = 1048576
    "IOReportLegendPublic" = Yes
    "IOMaximumSegmentByteCountWrite" = 1048576
    "NamespaceID" = 1
    "IOMaximumSegmentCountRead" = 256
    "IOMaximumSegmentCountWrite" = 256
    "IOStorageFeatures" = {"Unmap"=Yes,"Priority"=Yes,"Barrier"=Yes}
    "IOUnit" = 1
    "NamespaceUUID" = 0
    "Encryption" = Yes
    "Device Characteristics" = {"Serial Number"="---","Medium Type"="Solid State","Product Name"="APPLE SSD AP0512Q","Vendor Name"="","Product Revision Level"="1161.40."}
    "IOMaximumBlockCountRead" = 256
    "IOCFPlugInTypes" = {"AA0FA6F9-C2D6-457F-B10B-59A13253292F"="NVMeSMARTLib.plugin"}
    "IOMinimumSegmentAlignmentByteCount" = 4096
    "IOMaximumByteCountRead" = 1048576
    "IOMaximumByteCountWrite" = 1048576
    "device-type" = "Generic"
    "EmbeddedDeviceTypeRoot" = Yes
    "Physical Block Size" = 4096
    "Protocol Characteristics" = {"Physical Interconnect"="Apple Fabric","Physical Interconnect Location"="Internal"}
    "IOReportLegend" = ({"IOReportGroupName"="NVMe","IOReportChannels"=((6082504312848663127,6442450945,"Tier0 BW Scale Factor"),(6082504312865440343,6442450945,"Tier1 BW Scale Factor"),(6082504312882217559,6442450945,"Tier2 BW Scale Factor"),(6082504312898994775,6442450945,"Tier3 BW Scale Factor")),"IOReportChannelInfo"={"IOReportChannelUnit"=0},"IOReportSubGroupName"="BW Limits"},{"IOReportGroupName"="NVMe","IOReportChannels"=((6084209303804800357,6442450945,"Total time elapsed"),(6082504312848654368,6442450945,"Tier0 Throttle Time"),(6082504312865431584,6442450945,"Tier1 Throttle Time"),(6082504312882208800,6442450945,"Tier2 Throttle Time"),(6082504312898986016,6442450945,"Tier3 Throttle Time")),"IOReportChannelInfo"={"IOReportChannelUnit"=0},"IOReportSubGroupName"="Time weighted throttle statistics"})
    "ThermalThrottlingSupported" = Yes
    "NVMe SMART Capable" = Yes
    "Queue Depth Counters" = {"QueueDepths"=(277466,57709,13737,7000,4380,3403,2733,2057,1387,997,732,545,397,318,227,148,32,23,17,24,20,19,20,14,13,14,11,10,10,9,9,9,12,10,12,11,13,11,9,10,10,9,9,7,7,6,6,4,3,4,2,2,4,2,2,2,1,1,1,2,3,4,10,0)}

`smartctl` output (if the MacBook runs on battery SMART queries take ages, most probably related to energy savings):

    bash-3.2# smartctl -q noserial -a /dev/disk0
    smartctl 7.0 2018-12-30 r4883 [Darwin 20.1.0 x86_64] (local build)
    Copyright (C) 2002-18, Bruce Allen, Christian Franke, www.smartmontools.org
    
    === START OF INFORMATION SECTION ===
    Model Number:                       APPLE SSD AP0512Q
    Firmware Version:                   1161.40.
    PCI Vendor/Subsystem ID:            0x106b
    IEEE OUI Identifier:                0x000000
    Controller ID:                      0
    Number of Namespaces:               3
    Local Time is:                      Fri Nov 20 23:26:19 2020 CET
    Firmware Updates (0x02):            1 Slot
    Optional Admin Commands (0x0004):   Frmw_DL
    Optional NVM Commands (0x0004):     DS_Mngmt
    Maximum Data Transfer Size:         256 Pages
    
    Supported Power States
    St Op     Max   Active     Idle   RL RT WL WT  Ent_Lat  Ex_Lat
     0 +     0.00W       -        -    0  0  0  0        0       0
    
    === START OF SMART DATA SECTION ===
    SMART overall-health self-assessment test result: PASSED
    
    SMART/Health Information (NVMe Log 0x02)
    Critical Warning:                   0x00
    Temperature:                        26 Celsius
    Available Spare:                    100%
    Available Spare Threshold:          99%
    Percentage Used:                    0%
    Data Units Read:                    745,421 [381 GB]
    Data Units Written:                 766,941 [392 GB]
    Host Read Commands:                 11,365,081
    Host Write Commands:                5,750,127
    Controller Busy Time:               0
    Power Cycles:                       163
    Power On Hours:                     5
    Unsafe Shutdowns:                   2
    Media and Data Integrity Errors:    0
    Error Information Log Entries:      0
   
   Read Error Information Log failed: NVMe admin command:0x02/page:0x01 is not supported

### USB4 / Thunderbolt

The new M1 Macs are advertised as USB4/TB3 capable. Since almost everything is optional with USB4 this means with those three "Apple Silicon" Macs for now:

  * Thunderbolt 3 compliant
  * maximum SuperSpeed 10Gbps USB transfer speeds

A Mac Mini teardown revealed that the TB controller is inside the M1 SoC and there are two [JHL8040R](https://ark.intel.com/content/www/de/de/ark/products/186251/intel-jhl8040r-thunderbolt-4-retimer.html) TB retimer chips soldered next to the USB-C ports. While these chips are TB4 capable Apple only implements TB3, most probably not all of the needed requirements for TB4 could be met:

![](https://heise.cloudimg.io/width/3092/q70.png-lossy-70.webp-lossy-70.foil1/_www-heise-de_/imgs/18/3/0/0/1/5/6/8/thunderbolt4-comparison-chart-2f0aae9c6d8e08d7.jpg)

Quick check with IPoverThunderbolt using `iperf3` with default settings (MTU 1500 bytes): ARM -> Intel results in 17 Gbit/sek (cpu0 almost maxing out on the Intel MacBook for `kernel_task` handling the IRQ processing). In the other direction we're at 15 Gbit/sek, this time cpu5 and cpu6 (both performance cores) busy on the ARM MacBook. Since `iperf3` is `x86_64` and therefore executed by Rosetta2 the results should be taken with a grain of salt but for me it's already 'fast enough' (quick check with MTU 9000 ended up with slightly *lower* numbers for whatever reason).



## Software

### Universal Binaries

Almost every binary in macOS 11 is now an [Universal Binary](https://en.wikipedia.org/wiki/Universal_binary#Universal_2) in the sense that it contains both `x86_64` code and `arm64e`. Notable exceptions: *Rosetta 2 Updater.app* is `arm64e` only and */System/Library/Frameworks/OpenCL.framework* is `x86_64` only (for a list of apps use `Utilities:System Information.app` or `system_profiler SPApplicationsDataType`)

Let's look at an utilitiy that hasn't been upgraded since ages (bash on macOS is still on version 3.2.57). The size of `/bin/bash` in 10.15.7 is 623472 bytes. In 11.0.1 it's 1296640 bytes and the `arm64e` portion is slightly larger:

    mac-tk-air:~ tk$ ls -la /bin/bash
    -r-xr-xr-x  1 root  wheel  1296640  1 Jan  2020 /bin/bash
    mac-tk-air:~ tk$ lipo -detailed_info /bin/bash
    Fat header in: /bin/bash
    fat_magic 0xcafebabe
    nfat_arch 2
    architecture x86_64
        cputype CPU_TYPE_X86_64
        cpusubtype CPU_SUBTYPE_X86_64_ALL
        capabilities 0x0
        offset 16384
        size 627088
        align 2^14 (16384)
    architecture arm64e
        cputype CPU_TYPE_ARM64
        cpusubtype CPU_SUBTYPE_ARM64E
        capabilities PTR_AUTH_VERSION USERSPACE 0
        offset 655360
        size 641280
        align 2^14 (16384)

When looking at other binaries sometimes Intel is larger and sometimes ARM. So let's check a bunch of applications below `/System/Applications/` <span id="a1">[ [1] ](#f1)</span>:

| App | x86_64 | arm64e |
| -------- | -----: | -----: |
| App Store.app | 4655200 | 4685344 |
| Automator.app | 467008 | 463680 |
| Books.app | 1236560 | 1177872 |
| Calculator.app | 234544 | 248464 |
| Calendar.app | 3492016 | 3367552 |
| Chess.app | 293808 | 290688 |
| Contacts.app | 1459104 | 1439728 |
| Dictionary.app | 381088 | 377792 |
| FaceTime.app | 1625440 | 1596064 |
| FindMy.app | 3315504 | 3364992 |
| Font Book.app | 1139408 | 1095856 |
| Home.app | 796048 | 786912 |
| Image Capture.app | 76256 | 75488 |
| Launchpad.app | 55280 | 38704 |
| Mail.app | 4892320 | 4739744 |
| Maps.app | 18705248 | 18109472 |
| Messages.app | 306784 | 305296 |
| Mission Control.app | 55328 | 38720 |
| Music.app | 31003248 | 29141344 |
| News.app | 7074800 | 6928288 |
| Notes.app | 4942880 | 4857760 |
| Photo Booth.app | 772384 | 748224 |
| Photos.app | 18105216 | 17718384 |
| Podcasts.app | 4914992 | 4901568 |
| Preview.app | 2706848 | 2554464 |
| QuickTime Player.app | 1664432 | 1552528 |
| Reminders.app | 3386624 | 3443552 |
| Siri.app | 55392 | 38800 |
| Stickies.app | 181968 | 180432 |
| Stocks.app | 448432 | 457920 |
| System Preferences.app | 312544 | 292800 |
| TV.app | 22062464 | 20898704 |
| TextEdit.app | 188112 | 186464 |
| Time Machine.app | 55968 | 39328 |
| VoiceMemos.app | 2999936 | 2896928 |
| Summary: | 144063184 | 139039856 |

Intel binaries are roughly 4% larger. Another check for system frameworks <span id="a2">[ [2] ](#f2)</span> shows the opposite: here the `arm64e` framework binaries are 4% larger than their Intel counterparts so it's save to assume that the binary portions are roughly identical in size.

It's also save to assume that the storage requirements for macOS 11 being an 'Universal Binary' release is a bit larger than `x86_64` only former macOS versions. But Apple started with transparent file compression back in OS X 10.6 by tweaking HFS+ in some way. And with APFS today not much has changed and we benefit from compressed data on disk (the `afsctool` isn't on latest version and therefore reports 'HFS+ compression' while in reality we're talking APFS instead)

    mac-tk-air:~ tk$ afsctool -v /bin/bash 
    /bin/bash:
    File is HFS+ compressed.
    File content type: public.unix-executable
    File size (uncompressed data fork; reported size by Mac OS 10.6+ Finder): 1296640 bytes / 1.3 MB (megabytes) / 1.2 MiB (mebibytes)
    File size (compressed data fork - decmpfs xattr; reported size by Mac OS 10.0-10.5 Finder): 695527 bytes / 696 KB (kilobytes) / 680 KiB (kibibytes)
    File size (compressed data fork): 695543 bytes / 696 KB (kilobytes) / 680 KiB (kibibytes)
    Compression savings: 46.4%
    Number of extended attributes: 0
    Total size of extended attribute data: 0 bytes
    Approximate overhead of extended attributes: 536 bytes
    Approximate total file size (compressed data fork + EA + EA overhead + file overhead): 697120 bytes / 697 KB (kilobytes) / 681 KiB (kibibytes)



(TBC)

  * information about battery and connected USB-C chargers are available through the AppleSmartBattery object (compare with `ioreg -l`)
  * `pmset -g log`



Footnotes
=========

1. <span id="f1"></span>`for app in /System/Applications/*.app ; do Sizes=$(lipo -detailed_info "${app}/Contents/MacOS/$(basename "${app}" .app)" 2>/dev/null | awk -F" " '/size/ {print $2}'); set $Sizes; echo -e "${app##*/}\t$1\t$2"; done >/Users/tk/app-sizes.txt` [back](#a1)
2. <span id="f2"></span>`find /System/Library/Frameworks -name "*dylib" | while read ; do Sizes=$(lipo -detailed_info ${REPLY} 2>/dev/null | awk -F" " '/size/ {print $2}'); [ -z $Sizes ] || set $Sizes; echo -e "${REPLY##*/}\t$1\t$2"; done >/Users/tk/framework-sizes.txt` [back](#a2)