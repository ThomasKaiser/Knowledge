# Quick Preview of NanoPi R5S

**(Work in progress)**

The board obviously wants to be used as router/firewall though other use cases are possible as well since USB3 ports, HDMI and I2S are exposed so this device could also be used as a NAS (for example with a [5-port SATA card in the M.2 slot](https://forum.odroid.com/viewtopic.php?f=212&t=44318)) or a NAS/mediaplayer combo or some 'smart' video surveillance thingy.

But its main feature are of course the two 2.5GbE ports and FriendlyELEC charges [between $59 for the bare board with 2 GB RAM + 8GB eMMC and 85 bucks for 4/16GB with metal enclosure](https://www.friendlyelec.com/index.php?route=product/product&path=69&product_id=287). Unfortunately FriendlyELEC's EU distributor is not going to list new products due to missing CE certification so we ended up ordering two 2GB variants with enclosure directly from China and as such the $150 total turned into ~250€ here in the EU including customs, VAT and DHL shipping/processing.

![](../media/R5S_front_back.jpg)

## Overview

  * SoC: Rockchip RK3568 quad-core Cortex-A55 processor *up to* 2.0 GHz with Arm Mali-G52 MP2 GPU, 0.8 TOPS AI accelerator, 4Kp60 H.265/H.264/VP9 video decoder, 1080p60 H.264/H.265 video encoder
  * System Memory: 2GB or 4GB LPDDR4X
  * 8GB or 16GB eMMC flash for OS
  * Key M socket for M.2 2280 (PCIe Gen2 x1) NVMe SSD support
  * SDXC compatible Micro-SD card slot
  * HDMI 2.0 port up to 4Kp60, or 1080p120
  * 2 x 2.5GbE RJ45 ports (via Realtek RTL8125BG PCIe controller)
  * 1 x Gigabit Ethernet RJ45 port (RGMII attached with RTL8211F PHY)
  * 2 x USB3 SuperSpeed ports
  * 16-pin 1.27mm pitch GPIO connector with 1 x SDIO 3.0, 1 x I2S (2 x SDO, 3 x SDI)
  * 12-pin 0.5mm pitch FPC connector with up to 1 x SPI, up to 3 x UARTs, up to 4 x PWMs, up to 8 x GPIOs
  * 3-pin debug UART header, 3.3V level, 1500000bps
  * Misc: Mask key for eMMC flash update, RTC battery connector, 5V fan header, 4 x user LEDs
  * Power Supply: 5V/9V/12V USB-C port (USB PD support)
  * Dimensions: PCB 90 x 62 mm, enclosure 94.5 x 68 x 30 mm
  * Weight: 57.5 grams without the case, 260 grams with the metal enclosure

More information can be found in [FriendlyELEC's wiki](https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R5S).

## RK3568

At the heart of the device is Rockchip's RK3568 SoC in the more recent RK3568B2 variant which is a plastic package version of original RK3568 that came in a metal-can type packaging. Rockchip's BSP kernel seems to allow for differentiation since all those boards using the plastic RK3568B2 variant (Banana Pi R2 Pro, [Mrkaio AIO-M68S](https://archive.ph/oibqp), NanoPi R5S, ODROID-M1, and Radxa ROCK 3A) show up with `rockchip-cpuinfo cpuinfo: SoC: 35682000` in `dmesg` output while older Firefly RK3568-ROC-PC with the metal variant reads `35681000`.

Asides packaging they should be the same: RK3568(B2) is a quad-core Cortex-A55 SoC said to be made in a '22nm process' and clocking *up to* 2 GHz.

The CPU cores support the following extensions: 'fp asimd evtstrm aes pmull sha1 sha2 crc32 atomics fphp asimdhp cpuid asimdrdm lrcpc dcpop asimddp'. The topology looks like this:

    CPU sysfs topology (clusters, cpufreq members, clockspeeds)
                     cpufreq   min    max
     CPU    cluster  policy   speed  speed   core type
      0        0        0      408    1992   Cortex-A55 / r2p0
      1        0        0      408    1992   Cortex-A55 / r2p0
      2        0        0      408    1992   Cortex-A55 / r2p0
      3        0        0      408    1992   Cortex-A55 / r2p0

So we have a single CPU cluster and the maximum cpufreq OPP reading 1992 MHz needs to be taken with a huge grain of salt since PVTM is at work here ([discussed more in detail with ROCK 5B recently](https://github.com/ThomasKaiser/Knowledge/blob/master/articles/Quick_Preview_of_ROCK_5B.md#pvtm)). We've seen RK3568 being [clocked down to just 1840 MHz while cpufreq driver still reports 1992 MHz](https://forum.odroid.com/viewtopic.php?p=350782#p350782). It has to be seen whether on those lower quality silicon we can manually 'overvolt' the cores slightly to get advertised clockspeeds in exchange for some more generated heat.

Performance is a bit lower compared to similar quad-core A55 designs (like S905X3 on ODROID C4/HC4 that clocks with real 2100 MHz) but it's ok-ish and especially memory intensive tasks benefit from the much better memory performance compared to its predecessor Cortex-A53. 

Since already mentioning ROCK 5B: when comparing with the 'little' A55-cluster in RK3588 [the difference is huge both in performance and consumption](https://forum.radxa.com/t/rock-5b-debug-party-invitation/10483/61?u=tkaiser) caused by memory access and process node. 

## Powering

According to schematics a Fairchild FUSB302 USB PD controller is used and just as in ROCK 5B's case defaulting only to 5V/9V/12V ([which we changed almost immediately there](https://forum.radxa.com/t/powering-rock-5b/10759/3?u=tkaiser)). But [unlike ROCK 5B](https://github.com/radxa/kernel/blob/5e6d32859dfb73c1cfeefcc8074282480219caab/arch/arm64/boot/dts/rockchip/rk3588-rock-5b.dts#L641-L714) the respective I2C4 node is missing in DT settings to configure and monitor the chip.

## Consumption

## LEDs

    root@FriendlyWrt:/sys/class/leds# ls -la
    total 0
    drwxr-xr-x  2 root root 0 Aug  4  2017 .
    drwxr-xr-x 76 root root 0 Aug  4  2017 ..
    lrwxrwxrwx  1 root root 0 Aug  4  2017 lan1_led -> ../../devices/platform/gpio-leds/leds/lan1_led
    lrwxrwxrwx  1 root root 0 Aug  4  2017 lan2_led -> ../../devices/platform/gpio-leds/leds/lan2_led
    lrwxrwxrwx  1 root root 0 Aug  4  2017 mmc2:: -> ../../devices/platform/fe310000.sdhci/leds/mmc2::
    lrwxrwxrwx  1 root root 0 Aug  4  2017 sys_led -> ../../devices/platform/gpio-leds/leds/sys_led
    lrwxrwxrwx  1 root root 0 Aug  4  2017 wan_led -> ../../devices/platform/gpio-leds/leds/wan_led

`mmc2` is a phantom but the other 4 nodes represent real leds: `sys_led` is the outer left red led defaulting to `heartbeat`, the other three are green and set to `none` in FriendlyELEC's Ubuntu image and `netdev` with FriendlyWRT with [these pin settings](https://github.com/friendlyarm/kernel-rockchip/blob/8625799dde7abbc85ebc4ea9b549a813facd5148/arch/arm64/boot/dts/rockchip/rk3568-nanopi5-rev01.dts#L225-L245). 

Other possible `trigger` values as follows: `[none] rc-feedback rfkill-any rfkill-none kbd-scrolllock kbd-numlock kbd-capslock kbd-kanalock kbd-shiftlock kbd-altgrlock kbd-ctrllock kbd-altlock kbd-shiftllock kbd-shiftrlock kbd-ctrlllock kbd-ctrlrlock test_ac-online test_battery-charging-or-full test_battery-charging test_battery-full test_battery-charging-blink-full-solid test_usb-online mmc0 mmc2 timer oneshot heartbeat gpio cpu cpu0 cpu1 cpu2 cpu3 default-on panic`

## PCIe

RK3568 features two Gen3 lanes and a single Gen2 lane. To each of the Gen3 lanes is a RTL8125BG 2.5GbE NIC attached (making use of [bifurcation](https://github.com/friendlyarm/kernel-rockchip/blob/8625799dde7abbc85ebc4ea9b549a813facd5148/arch/arm64/boot/dts/rockchip/rk3568-nanopi5-rev01.dts#L193-L222)) and the Gen2 lane is routed to the M.2 key M slot. One could argue that the M.2 slot would've be better Gen3 (twice the bandwidth, lower latency) since the RTL8125BG while attached to the PCIe 3.0 controller are set to Gen2 link speed anyway: `Speed 5GT/s (ok), Width x1 (ok)`. But maybe the Gen3 lanes even only with Gen2 link speed show lower latency compared to the PCIe 2.1 controller?

Further tests might show...

## USB

RK3568 relies on [Naneng Micro's USB3.0/PCIE2/SATA3 Combo PHY](http://www.nanengmicro.com/en/combo-phy/) which results in SATA, USB3 and PCIe Gen2 signals all being multiplexed:

![](../media/RK3568-multiplexed-sata-usb-3.0-pcie.jpg)

In R5S' case we're ending up with 1 x PCIe and 2 x USB3 and no SATA at all (or only if you slap a SATA controller into the M.2 slot). The right USB3 slot can be turned into OTG/MASKROM mode so with an A-to-A USB cable (violating the USB specs) you can turn R5S into an USB gadget or [flash the eMMC via USB](https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R5S#Option_3:_Install_OS_via_USB) .

1.4A overcurrent protection

`/sys/class/typec/` empty due to missing I2C4 device-tree node

## SD card and eMMC

https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R5S#The_Boot_order_between_eMMC_and_SD_card

The 8GB eMMC on both our R5S was prepopulated with FriendlyWRT based on Rockchip's 5.10.66 BSP kernel ([not to be confused with 5.10 LTS from kernel.org](https://www.cnx-software.com/2022/01/09/rock5-model-b-rk3588-single-board-computer/#comment-589709)).

## Networking

    driver: r8125
    version: 9.009.00-NAPI
    firmware-version:
    expansion-rom-version:
    bus-info: 0001:11:00.0
    supports-statistics: yes
    supports-test: no
    supports-eeprom-access: no
    supports-register-dump: yes
    supports-priv-flags: no


    root@FriendlyWrt:~# modinfo r8125
    filename:       /lib/modules/5.10.110/r8125.ko
    version:        9.009.00-NAPI
    license:        GPL
    description:    Realtek RTL8125 2.5Gigabit Ethernet driver
    author:         Realtek and the Linux r8125 crew <netdev@vger.kernel.org>
    srcversion:     569E79BC38A800B06207F7F
    alias:          pci:v000010ECd00003000sv*sd*bc*sc*i*
    alias:          pci:v000010ECd00008162sv*sd*bc*sc*i*
    alias:          pci:v000010ECd00008125sv*sd*bc*sc*i*
    depends:        
    name:           r8125
    vermagic:       5.10.110 SMP mod_unload modversions aarch64
    parm:           speed_mode:force phy operation. Deprecated by ethtool (8). (uint)
    parm:           duplex_mode:force phy operation. Deprecated by ethtool (8). (uint)
    parm:           autoneg_mode:force phy operation. Deprecated by ethtool (8). (uint)
    parm:           advertising_mode:force phy operation. Deprecated by ethtool (8). (uint)
    parm:           aspm:Enable ASPM. (int)
    parm:           s5wol:Enable Shutdown Wake On Lan. (int)
    parm:           s5_keep_curr_mac:Enable Shutdown Keep Current MAC Address. (int)
    parm:           rx_copybreak:Copy breakpoint for copy-only-tiny-frames (int)
    parm:           use_dac:Enable PCI DAC. Unsafe on 32 bit PCI slot. (int)
    parm:           timer_count:Timer Interrupt Interval. (int)
    parm:           eee_enable:Enable Energy Efficient Ethernet. (int)
    parm:           hwoptimize:Enable HW optimization function. (ulong)
    parm:           s0_magic_packet:Enable S0 Magic Packet. (int)
    parm:           tx_no_close_enable:Enable TX No Close. (int)
    parm:           enable_ptp_master_mode:Enable PTP Master Mode. (int)
    parm:           disable_pm_support:Disable PM support. (int)
    parm:           debug:Debug verbosity level (0=none, ..., 16=all) (int)




4a:8d:7a:80:67:1f with FriendlyWRT

Wi-Fi `iwconfig wlan0 power on`

LAA MAC addresses with Ubuntu

  * eth0: 5e:21:6a:4e:7c:1f
  * eth1: 62:21:6a:4e:7c:1f
  * eth2: 66:bb:4e:d4:90:c1

16K firmware BLOB for the PCIe 3 PHY: https://github.com/friendlyarm/kernel-rockchip/commit/c056c33a88c05d64bfac3687e6fbab978c7d1f57#diff-8c17abb46c49a69aaf02070eb70b2b5cc0f0e785376862da61ce948a21eb628e

## RTC

A HYM8563TS low-power RTC is on the board, expecting a backup current of 0.25μA TYP (VDD=3.0V) from a user supplied backup battery connected to the 2 pin 1.27/1.25mm Molex 53398-0271 connector.

Quick check for `/dev/rtc0` succeeded: `hwclock -r -f /dev/rtc0 -> 2022-07-13 22:53:46.165661+02:00`.

## Audio

[RK809](https://rockchip.fr/RK809%20datasheet%20V1.01.pdf)

## SPI NOR flash socket

You could solder yourself an SPI NOR flash to the board but since there's already eMMC this doesn't make much sense. Most probably it's for large custom orders w/o eMMC so the bootloader can be written to the flash to let the board boot from USB, NVMe SSD or network.

## Enclosure

![](../media/R5S_enclosure.jpg)

Opening for a Wi-Fi antenna

## Software

FriendlyElEC's OS images (both Ubuntu and FriendlyWRT) use Rockchip MiniLoader and an Android partition scheme which I'm not familiar with at all:

    179        0    7634944 mmcblk0
    179        1       4096 mmcblk0p1
    179        2       4096 mmcblk0p2
    179        3       4096 mmcblk0p3
    179        4      16384 mmcblk0p4
    179        5      40960 mmcblk0p5
    179        6      32768 mmcblk0p6
    179        7      32768 mmcblk0p7
    179        8     376832 mmcblk0p8
    179        9    7114735 mmcblk0p9

    overlay / overlay rw,relatime,lowerdir=/root,upperdir=/data/root,workdir=/data/work 0 0


No `extlinux.conf`, no boot script, no way to simply exchange the kernel. `/proc/cmdline` looks like this when booting from SD card: `storagemedia=sd androidboot.storagemedia=sd androidboot.mode=normal androidboot.dtbo_idx=1 androidboot.verifiedbootstate=orange earlycon=uart8250,mmio32,0xfe660000 console=ttyFIQ0 coherent_pool=1m rw root=/dev/mmcblk0p8 rootfstype=ext4 data=/dev/mmcblk0p9 consoleblank=0 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory swapaccount=1`



https://github.com/friendlyarm/kernel-rockchip/commits/nanopi5-v5.10.y_opt

https://pastebin.com/PPmBMNaM

https://github.com/mj22226/openwrt/commit/ef237a318622d4b51536b3d2a456c33d03502319

## FriendlyWRT

https://github.com/friendlyarm/friendlywrt/blob/13c5704db185a1f2a8ae9e53921da8e93f2f2130/target/linux/rockchip/armv8/base-files/etc/hotplug.d/net/40-net-smp-affinity




Overclocking: OPP table adjustment, from 1.9 to 2.39 GHz:

http://ix.io/4hkG
http://ix.io/4hkN
http://ix.io/4hkW
http://ix.io/4hl3
http://ix.io/4hls 
http://ix.io/4hqP
http://ix.io/4hS3 



