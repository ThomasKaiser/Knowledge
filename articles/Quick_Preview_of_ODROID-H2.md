# Quick Preview of ODROID H2

![](https://dn.odroid.com/ODROID-H2/s_ODROID-H2.jpg)

The following is not a real review since I only had remote access to three H2 running Ubuntu Bionic. It's more of a hardware overview focused on server centric use cases.

## Overview

This new ODROID board is 110x110x43mm in size not following any established form factor. For full details including enclosure variants see [Hardkernel's announcement thread](https://forum.odroid.com/viewtopic.php?f=29&t=32536). Here are just some quick specs:

* 2.4/2.5Ghz Quad-core Celeron J4105 (Gemini Lake / 14nm) with 4MiB Cache
* no Hyper-Threading but AES-NI and virtualization ready: VT-d and VT-x with Extended Page Tables (EPT)
* Dual-channel DDR4-PC19200 memory controller (2400MT/s), not ECC capable
* two SO-DIMM slots for a total of 32GiB RAM (tested, according to Intel [only 8GB are possible](https://ark.intel.com/products/128989/Intel-Celeron-J4105-Processor-4M-Cache-up-to-2-50-GHz-))
* PCIe Gen2 x4 available on one M.2 key M slot only providing PCIe
* 2 x RTL8111G based Gbit Ethernet ports each behind an own PCIe lane
* 2 x native SATA 3.0
* 2 x USB3 type A ports
* 2 x USB2 type A ports (Gemini Lake has no EHCI controller any more so these are also provided by the xHCI controller)
* SSE4.2 accelerator (SMM, FPU, NX, MMX, SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AES) 
* Intel UHD Graphics (Gen9.5) 600 (GT1) 700Mhz
* HDMI 2.0 and DP 1.2 multiple video outputs

`/proc/cpuinfo`, `lsusb -v`, `lspci -vv` and `lshw` output available [here](http://ix.io/1qbj). The most important question 'How much will this thingy cost?' is not yet answered by Hardkernel. But since they want the H2 being shipped at the end of November 2018 we'll know soon.

## CPU and memory performance

The ['Goldmont Plus'](https://en.wikichip.org/wiki/intel/microarchitectures/goldmont_plus) Celeron J4105 on this board is able to clock up to 2490 MHz with one active CPU core and  2396 MHz with more cores busy and is then slightly faster than the 2016 Goldmont Pentiums/Celerons (N420x/J420x) and +30% faster than RK3399 designs or ODROID XU4/HC1/HC2. Detailed performance scores will appear soon at the bottom of [sbc-bench results table](https://github.com/ThomasKaiser/sbc-bench/blob/master/Results.md). According to Hardkernel their passive heatsink is sufficient for sustained high loads (again see their [announcement thread](https://forum.odroid.com/viewtopic.php?f=29&t=32536))

Please note that performance relies on memory configuration. For full performance two DDR4-PC19200 (2400MT/s) DIMMs of same size are needed (Gemini Lake can not cope with faster DDR4-PC21300 modules). If you use only one module depending on the use case performance can decrease by 20% or even more. See Hardkernel's FFmpeg transcoding test:

![](https://dn.odroid.com/ODROID-H2/transcoding_rate.png)

It should also be noted that when the graphics engine is active maximum cpufreq is only 2.2 GHz any more. For further performance tests you can simply rely on the Internet since all that matters is the CPU type and benchmarks done with e.g. an ASRock J4105 will apply here too.

## Storage performance

I was only able to test with 3 different SSDs using `iozone -e -I -a -s 100M -r 4k -r 16k -r 512k -r 1024k -r 16384k -i 0 -i 1 -i 2 ; sleep 300 ; iozone -e -I -a -s 2000M -r 16384k -i 0 -i 1 -i 2` ([sleeping 300 seconds in between for a reason](https://forum.armbian.com/topic/8097-nanopi-m4-performance-and-consumption-review/?do=findComment&comment=61783))

    SATA Samsung 860 EVO 250GB / RVT01B6Q                         random    random
                  kB  reclen    write  rewrite    read    reread    read     write
              102400       4    72437    91850    94764    94543    44477    89849         
              102400      16   203068   237862   247603   249028   110745   236857         
              102400     512   451554   449136   471435   479800   438253   458596         
              102400    1024   434889   447961   441752   444064   457868   486368         
              102400   16384   459942   499154   534438   544153   533377   512175
             2048000   16384   510176   513427   543089   543100   539357   515008
              
    SATA TOSHIBA THNSNJ128GCSU / JURA0101                         random    random
                  kB  reclen    write  rewrite    read    reread    read     write
              102400       4    67806    84301    90314    89729    23800    83227         
              102400      16   183474   208377   231232   231709    77464   185007         
              102400     512   429386   461900   501213   493337   443476   449965         
              102400    1024   411651   425846   461589   509602   481078   420490         
              102400   16384   454119   470594   519969   530120   529409   473989                                
             2048000   16384   487956   488575   533611   529319   517535   488143
    
    NVMe WDS500G2X0C-00L350 / 101110WD                            random    random
                  kB  reclen    write  rewrite    read    reread    read     write
              102400       4   118800   166451   113793   114171    48654   161392                                                          
              102400      16   369882   475456   361790   360653   145655   463275                                                          
              102400     512  1181289  1222746  1404259  1418600  1253365  1275058                                                          
              102400    1024  1229515  1262795  1416458  1427279  1424415  1193388                                                          
              102400   16384  1032897  1406616  1667046  1687417  1688260  1333405                                                          
             2048000   16384  1339204  1355091  1691162  1692434  1691324  1351675

As expected the native SATA ports allow to exceed 540 MB/s (cheap single PCIe lane attached SATA controllers usually do not exceed 400 MB/s) and with NVMe SSDs almost 1.7 GB/s is possible. Please keep in mind that the PCIe lanes are only Gen2 here (Gen3 would be twice as fast) and that more and more NVMe SSDs appear with a Gen3 x2 interface. Those SSDs will then only negotiate a Gen2 x2 connection with the H2 and be severly bottlenecked. So only buy SSDs with an x4 interface!

I wasn't able to test USB3 storage performance but since it's Intel's own USB3 implementation we can predict results: USB Attached SCSI (UAS) capable and up to 400 MB/s on the 2 SuperSpeed ports. So most probably this board when running Linux with attached Seagate USB3 disks will run into troubles. With older kernels than 4.15 UAS needs to be blacklisted, starting with 4.15 [this fix](https://www.spinics.net/lists/linux-usb/msg162709.html) has been applied so Seagate disks while working then normally can not be queried about their SMART capabilities (again UAS blacklisting is the solution for external Seagate USB disks)

## PCIe configuration and use cases

The Intel CPU has 6 Gen2 PCIe lanes. Hardkernel exposes 4 of them on the M.2 key M socket at the bottom suitable for NVMe SSDs (no M.2 SATA SSDs!) or PCIe adapters/extenders. The two remaining PCIe lanes are used for the RTL8111G network chips.

When attaching NVMe SSDs two things are important: 

* since the SoC is only capable of Gen2 speeds you should only buy SSDs with an x4 interface (none of then Gen3 x2 variants since they will be really bottlenecked on Gen2 ports)
* fast NVMe SSDs can get quite hot so this needs to be addressed since otherwise the SSD will overheat and throttle performance (I tested with SSDs that got as slow as 30 MB/s sequential performance)

What if not using an NVMe SSD? Then there are plenty of opportunities to make use of the four PCIe lanes. Users could insert an [M.2 dual Gbit Ethernet card](https://www.dpie.com/m.2/m.2-ethernet-cardsswitches/innodisk-egpl-g201) for example. Unfortunately mounting holes for the shorter M.2 variants are missing so neither [this less expensive dual GbE card](https://shop.udoo.org/m-2-dual-ethernet-module-kit.html) is an option nor that [4 port SATA card](https://de.aliexpress.com/item/Gro-e-Q-M-2-PCIe-B-M-Schl-ssel-slot-zu-4-Port-SATA-6g/32894260145.html) (I hope Hardkernel will fix this on future PCB revisions).

But since it's PCIe x4 a simple adapter to turn M.2 into a regular x4 PCIe slot is of course also an option:

![](http://kaiser-edv.de/tmp/Eevikv/61fVgKsS2gL._SL1200_.jpg)

The 4 pin header that can be seen here wants to be fed with both 5V and 12V since this is basic powering requirement for standard PCIe cards. Fortunately on the H2 there are even two 4 pin headers providing exactly these two voltages (the SATA power ports). So in fact the H2 is a pretty good choice for adding normal PCIe cards as long as you DIY power cable and enclosure/fixation.

## Software support situation

Traditional ODROID users know from the ARM platform each and every board today needs own OS images (will hopefully change soon due to board makers starting to add some SPI NOR flash on their PCBs able to store a device specific bootloader and hardware description to then boot a device agnostic standard ARM OS image as we know it from the x86 platform. See also [EBBR](https://embedded-recipes.org/2018/talk/ebbr-standard-boot-for-embedded-platforms/))

With an Intel CPU this just works: take a somewhat decent Ubuntu x86 installer and everything works. Same with Windows but of course hardware needs drivers and this can be a problem. The OS image must be recent enough to deal with Gemini Lake so expect problems running old operating systems. For example Windows 7 will be a problem since with Gemini Lake even the USB 2.0 ports are provided by the xHCI controller and then without xHCI/USB3 driver support your USB boot stick won't boot ([possible workarounds](https://nucblog.net/2015/07/installing-windows-7-on-the-nuc5cpyh-or-nuc5ppyh/))

The chosen RealTek NICs can also be a problem [due to (lacking) driver support](https://www.anandtech.com/comments/13053/sapphire-unveils-fs-fp5v-amd-ryzen-embedded-mini-stx-motherboard). So if you want your H2 to be a small and energy efficient ESXi whitebox you'll need to add the driver package yourself (though no idea whether ESXi runs flawlessly with Gemini Lake CPUs).

Talking about virtualization... since the CPU has the appropriate features (VT-x, VT-d and VT-x/EPT) you can simply run one OS on top of another. See Hardkernel demonstrating Windows 10 running in Ubuntu via VirtualBox:

![](https://dn.odroid.com/ODROID-H2/181024_h2_vtx_virtualbox3.png)

## Networking

RealTek NICs have a pretty bad reputation, and a lot of people associate 'slow and unstable' with them. It has to be seen whether Hardkernel can work against this demonstrating stability and high performance (I suggested some tests but all of this stuff is somewhat time consuming so maybe we need to wait for later H2 owners doing the job)

Anyway: for basic stuff with decent driver support (IMO given at least with Linux and Windows) the NICs should do fine. I still hope for another board revision where at least one of the RTL8111G will be replaced by a NBase-T capable RTL8125 providing 2.5GbE networking out of the box. Or maybe a 'H2+' with two such RTL8125 (today with a PCIe adapter as shown above we could add rather cheap Tehuti TN9710P or Aquantia AQC107S based 10GbE/Nbase-T cards but onboard NBase-T would really be great and would make the H2 somewhat unique since GbE is quite slow for many use cases already and [two GbE NICs do **not** make up for a 2 Gbits/sec connection](https://www.cnx-software.com/2018/10/19/odroid-h2-intel-celeron-j4150-development-board/#comment-556717) even if an awful lot of people believe into this)

From a different point of view RealTek NICs instead of Intel ones can also be considered a feature. While paranoid people won't use any Intel equipped machine anyway due to the presence of the [Management Engine (ME)](https://libreboot.org/faq.html#intel) at least without Intel NICs part of the ME functionality won't work. One of the desired features -- Wake on LAN -- does work though with the RTL8111G next to the display connectors once the feature has been enabled (e.g. `sudo ethtool -s enp3s0 wol g` in Linux).

Below some more information wrt the NICs (`ip a` and `ethtool` output):

    2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether c4:82:4e:55:74:12 brd ff:ff:ff:ff:ff:ff
        inet 192.168.0.40/24 brd 192.168.0.255 scope global dynamic enp2s0
           valid_lft 43sec preferred_lft 43sec
        inet6 fe80::c682:4eff:fe55:7412/64 scope link 
           valid_lft forever preferred_lft forever
    3: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether c4:82:4e:55:74:13 brd ff:ff:ff:ff:ff:ff
        inet 192.168.0.41/24 brd 192.168.0.255 scope global dynamic enp3s0
           valid_lft 10sec preferred_lft 10sec
        inet6 fe80::c682:4eff:fe55:7413/64 scope link 
           valid_lft forever preferred_lft forever

    root@odroid-h2:~# ethtool -i enp2s0
    driver: r8169
    version: 2.3LK-NAPI
    firmware-version: rtl8168g-2_0.0.1 02/06/13
    expansion-rom-version: 
    bus-info: 0000:02:00.0
    supports-statistics: yes
    supports-test: no
    supports-eeprom-access: no
    supports-register-dump: yes
    supports-priv-flags: no
    
    root@odroid-h2:~# ethtool -S enp2s0
    NIC statistics:
         tx_packets: 6671
         rx_packets: 145775
         tx_errors: 0
         rx_errors: 0
         rx_missed: 0
         align_errors: 0
         tx_single_collisions: 0
         tx_multi_collisions: 0
         unicast: 9004
         broadcast: 135785
         multicast: 986
         tx_aborted: 0
         tx_underrun: 0
    
    root@odroid-h2:~# ethtool enp2s0
    Settings for enp2s0:
    	Supported ports: [ TP MII ]
    	Supported link modes:   10baseT/Half 10baseT/Full 
    	                        100baseT/Half 100baseT/Full 
    	                        1000baseT/Half 1000baseT/Full 
    	Supported pause frame use: No
    	Supports auto-negotiation: Yes
    	Supported FEC modes: Not reported
    	Advertised link modes:  10baseT/Half 10baseT/Full 
    	                        100baseT/Half 100baseT/Full 
    	                        1000baseT/Full 
    	Advertised pause frame use: Symmetric Receive-only
    	Advertised auto-negotiation: Yes
    	Advertised FEC modes: Not reported
    	Link partner advertised link modes:  10baseT/Half 10baseT/Full 
    	                                     100baseT/Half 100baseT/Full 
    	                                     1000baseT/Full 
    	Link partner advertised pause frame use: Symmetric
    	Link partner advertised auto-negotiation: Yes
    	Link partner advertised FEC modes: Not reported
    	Speed: 1000Mb/s
    	Duplex: Full
    	Port: MII
    	PHYAD: 0
    	Transceiver: internal
    	Auto-negotiation: on
    	Supports Wake-on: pumbg
    	Wake-on: g
    	Current message level: 0x00000033 (51)
    			       drv probe ifdown ifup
    	Link detected: yes
    
    root@odroid-h2:~# ethtool -d enp2s0
    RealTek RTL8168g/8111g registers:
    --------------------------------------------------------
    0x00: MAC Address                      c4:82:4e:55:74:12
    0x08: Multicast Address Filter     0x42000040 0x00400082
    0x10: Dump Tally Counter Command   0x6a638000 0x00000002
    0x20: Tx Normal Priority Ring Addr 0x6f0c4000 0x00000002
    0x28: Tx High Priority Ring Addr   0x00000000 0x00000000
    0x30: Flash memory read/write                 0x00000000
    0x34: Early Rx Byte Count                              0
    0x36: Early Rx Status                               0x00
    0x37: Command                                       0x0c
          Rx on, Tx on
    0x3C: Interrupt Mask                              0x803f
          SERR LinkChg RxNoBuf TxErr TxOK RxErr RxOK 
    0x3E: Interrupt Status                            0x0000
          
    0x40: Tx Configuration                        0x4f000f80
    0x44: Rx Configuration                        0x0002cf0e
    0x48: Timer count                             0x00000000
    0x4C: Missed packet counter                     0x000000
    0x50: EEPROM Command                                0x10
    0x51: Config 0                                      0x00
    0x52: Config 1                                      0xcf
    0x53: Config 2                                      0x3c
    0x54: Config 3                                      0x60
    0x55: Config 4                                      0x11
    0x56: Config 5                                      0x02
    0x58: Timer interrupt                         0x00000000
    0x5C: Multiple Interrupt Select                   0x0000
    0x60: PHY access                              0x00000000
    0x64: TBI control and status                  0x17ffff01
    0x68: TBI Autonegotiation advertisement (ANAR)    0xf70c
    0x6A: TBI Link partner ability (LPAR)             0x0000
    0x6C: PHY status                                    0xf3
    0x84: PM wakeup frame 0            0x00000000 0x00000000
    0x8C: PM wakeup frame 1            0x00000000 0x00000000
    0x94: PM wakeup frame 2 (low)      0x00000000 0x00000000
    0x9C: PM wakeup frame 2 (high)     0x00000000 0x00000000
    0xA4: PM wakeup frame 3 (low)      0x00000000 0x00000000
    0xAC: PM wakeup frame 3 (high)     0x00000000 0x00000001
    0xB4: PM wakeup frame 4 (low)      0xffffffff 0xd205c5e1
    0xBC: PM wakeup frame 4 (high)     0x00000000 0x00000000
    0xC4: Wakeup frame 0 CRC                          0x0000
    0xC6: Wakeup frame 1 CRC                          0x0000
    0xC8: Wakeup frame 2 CRC                          0x0000
    0xCA: Wakeup frame 3 CRC                          0x0000
    0xCC: Wakeup frame 4 CRC                          0x0000
    0xDA: RX packet maximum size                      0x4000
    0xE0: C+ Command                                  0x20e1
          VLAN de-tagging
          RX checksumming
    0xE2: Interrupt Mitigation                        0x5151
          TxTimer:       5
          TxPackets:     1
          RxTimer:       5
          RxPackets:     1
    0xE4: Rx Ring Addr                 0x6f0c5000 0x00000002
    0xEC: Early Tx threshold                            0x27
    0xF0: Func Event                              0x0000003f
    0xF4: Func Event Mask                         0x00000000
    0xF8: Func Preset State                       0x00000003
    0xFC: Func Force Event                        0x00000000