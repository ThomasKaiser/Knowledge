# Quick Preview of ROCK 5 ITX

![I/O ports view](../media/rock5-itx-1.jpg)

**(Work in progress)**

<details>
  <summary>More pictures</summary>

  ![Top view](../media/rock5-itx-2.jpg)
  
  ![Connector side view](../media/rock5-itx-3.jpg)

  ![SATA side view](../media/rock5-itx-4.jpg)

  ![PoE module](../media/rock5-itx-5.jpg)

  ![Board components](../media/rock5-itx-6.jpg)

  ![SD card, MIPI, Maskrom button, touchpad, eDP](../media/rock5-itx-7.jpg)

  ![M.2 key E, front panel header, audio jacks, RA620-1, ES8316](../media/rock5-itx-8.jpg)  

  ![Board with I/O Shield](../media/rock5-itx-9.jpg)

  ![Cardboard package](../media/rock5-itx-10.jpg)

</details>

## Overview

April 2024 Radxa started to send out developer samples of their RK3588 based [Mini-ITX](https://en.wikipedia.org/wiki/Mini-ITX) v1.11 board (for RK3588 basics and some notes about software support status see [Rock 5B](Quick_Preview_of_ROCK_5B.md)). Official documentation will once appear [here](https://docs.radxa.com/en/rock5/rock5itx) and for now you can see a block diagram [there](https://docs.radxa.com/en/assets/images/rock5itx-interface-overview-1266d3c0b4e745372a48a473d78c3cdc.webp)

The board measures 170x170mm in size as it follows the Mini-ITX standard with externally accessible connectors all on the 'back side' accompanied by an appropriate I/O shield. From left to right there's

  * 5.5/2.1 mm centre positive DC/IN: not wide input range but 12V only (directly connected to SATA power ports? TBC)
  * USB-C with OTG (USB 3.0/FullSpeed) and DisplayPort, no powering through this port
  * HDMI IN
  * RJ45 / 2.5GbE provided by RTL8125BG + USB 2.0 behind 4-port Terminus Inc. USB2 hub
  * another RJ45 / 2.5GbE provided by RTL8125BG + USB 2.0 behind same USB2 hub
  * 2 x USB3 behind [Genesys Logic GL3523 USB3 hub](https://linux-hardware.org/?id=usb:05e3-0620) and HDMI out (8K capable)
  * 2 x USB3 also behind the same GL3523 hub and HDMI out (4K capable, behind a Rockchip RK620-1 / Radxa RA620-1 DisplayPort-to-HDMI converter chip)
  * headphone/mic (via I2S attached [ES8316](http://everest-semi.com/pdf/ES8316%20PB.pdf))
  * [S/PDIF](https://en.wikipedia.org/wiki/S/PDIF)

On the next board side we find the following internal connectors:

  * front panel header (HDD LED, power LED, reset and power button)
  * audio header
  * USB2 header (two ports also behind same USB2 hub)
  * M.2 key E with PCIe Gen2 x1 or native SATA (switchable via device tree overlay)
  * 3-pin UART header
  * 2-pin recovery button header
  * TF card slot (SDR104 / UHS-I)
  * 2 x MIPI CSI (camera), 1 x MIPI DSI (LCD display)
  * TP (touch panel)
  * eDP (LCD display)
  * M.2 key M slot carrying PCIe Gen3 x2, able to fit 2230, 2242, 2260 and 2280 cards

On the remaining two board sides we find other internal connectors:

  * 4 power ports with 5V/12V for SATA devices
  * 4 x SATA 6Gbps ports (all behind an [ASMedia ASM1164](https://www.asmedia.com.tw/product/17fYQ85SPeqG8MT8/58dYQ8bxZ4UR9wG5) attached to RK3588 via PCIe Gen3 x2)
  * ATX power
  * header for an optional PoE module (already populated on my board)
  * 12V PWN enabled fan header

Other notable onboard components on *my* board include:

  * soldered 32GB "Samsung BJTD4R" HS400 eMMC module
  * 2 x 32Gb SK Hynix H58G56AK6B LPDDR5 modules for a total of 8GB DRAM
  * Rockchip RK806-1 PMIC
  * 128 Mb Winbond W25X20CL SPI NOR flash
  * a small I2C accessible HYM RTC chip
  * RTC battery holder
  * Maskrom key

## I/O capabilities

Some protocols on RK3588 are pinmuxed so the board designer has to decide between PCIe, USB3 and SATA in some cases. For a pinmuxing overview [see here](https://www.cnx-software.com/2021/12/16/rockchip-rk3588-datasheet-sbc-coming-soon/) and for a general RK3588 PCIe overview [see there](https://github.com/ThomasKaiser/Knowledge/blob/master/articles/Quick_Preview_of_ROCK_5B.md#pcie). On this board the seven available PCIe lanes are used as follows:

  * two of the Gen3 lanes are routed to the M.2 key M slot to be used with a SSD or M.2 storage/network adapters
  * the other two Gen3 lanes connect to the [ASM1164 SATA host controller](https://www.asmedia.com.tw/product/17fYQ85SPeqG8MT8/58dYQ8bxZ4UR9wG5) that provides the 4 SATA 6Gbps ports on the board. Talking about bandwidth PCIe Gen3 x2 and 4 x SATA at 6.0 Gbit/s with 8b/10b coding are a pretty close match as such not only four pieces of spinning rust but also fast SATA SSDs should be able to be accessed at full speed in parallel
  * two Gen2 lanes are used to attach the RTL8125BG Ethernet controllers
  * the last Gen2 lane is routed to the M.2 key E slot to be used with PCIe peripherals like a Wi-Fi card or when switched to SATA mode as another (and this time SoC native) SATA port. Making use of SATA requires a passive adapter like Radxa's 'M.2 E Key to SATA Adapter' that unfortunately seems to be sold out everywhere
  * all four USB3 receptacles are behind a GL3523 USB3 hub and as such have to share bandwidth
  * the same goes for the four USB2 ports (two receptables and two on headers) that are behind an USB2 hub
  * The USB3 OTG port available at the USB-C receptacle can be used for ADB by default but it should also be possible to use it as USB3 host port via a device-tree overlay
  * the TF card slot is UHS-I/SDR104 capable
  * the eMMC interface utilizes HS400 mode

## Display capabilities

Disclaimer: not my area since I usually operate SBC headless as such just a quick list copy&pasted from Radxa:

  * DisplayPort Alt mode available at the USB-C port (4Kp60)
  * HDMI 8Kp60 available native by the SoC
  * HDMI 4K available via Radxa RA620-1 DisplayPort-to-HDMI converter 
  * eDP FPC connector for 4Kp60 LCD panel
  * up to 2 x four-lane MIPI DSI connectors

That makes for six displays in total but according to Radxa only four can be used concurrently.

## Video input capabilities

  * HDMI up to 4Kp60
  * up to 2 x four-lane MIPI CSI connectors (switching between CSI and DSI can be done by a device-tree overlay)




(stub from here on)





SK Hynix 32Gb LPDDR5 modules should be faster than the LPDDR4X on Rock 5B (4224 vs. 5472 MT/s)
