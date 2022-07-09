# ARMv8 SoCs with PCIe support

## Multi purpose SoCs

* Allwinner H6 (1 x Gen2 x1). Note: [quirky controller not able to work with -- non existing -- generic drivers](https://linux-sunxi.org/Mainlining_Effort#cite_note-h6-pcie-4). As of march 2020 thanks to @Icenowy a [possible workaround using virtualization](https://forum.armbian.com/topic/13529-a-try-on-utilizing-h6-pcie-with-virtualization/) exists.
* Amlogic S922X/A311D (1 x Gen2 x1 muxed with USB3). Note: Gen2 x1 speed [seems reasonable](https://www.cnx-software.com/2019/05/14/khadas-vim3-amlogic-s922x-board-m-2-nvme-ssd-wifi-5-bluetooth-5/#comment-562858)
* Amlogic S905X3/S905X4 (1 x Gen2 x1 muxed with USB3). Note: Gen2 x1 speed [seems reasonable](https://forum.odroid.com/viewtopic.php?f=29&t=40609)
* Broadcom BCM2711 (1 x Gen2 x1)
* MediaTek MT7622 (2 x Gen2 x1)
* Nvidia Jetson AGX (5 x Gen4 (16GT/s), 1x8, 1x4, 1x2, 2x1 Root port and endpoint)
* Nvidia Jetson AGX Orin (up to 22 x Gen4 (16GT/s), up to 2x8, 1x4, 2x1 Root Port & Endpoint)
* Nvidia Jetson Nano (1 x Gen2 x1, 1 x Gen2 x4)
* Nvidia Jetson Orin NX (3 x Gen4 x1 + 1 x Gen4 x4)
* Nvidia Jetson TX1 (1 x Gen2 x1, 1 x Gen2 x4)
* Nvidia Jetson TX2 (2 x Gen2 x4)
* NXP i.MX8 (up to 1 x Gen3 x2 or 2 x Gen3 x1, another x1 multiplexed with SATA)
* Phytium FT-1500A (2 x Gen3 x16 or 4 x Gen3 x8, or 2 x Gen3 x8, 1 x Gen3 x16)
* Phytium FT-2000A (1 x Gen3 x8 or 2 x Gen3 x4)
* Phytium FT-2000/D2000 (2 x Gen3 x1 and 2 x Gen3 x16 or 4 x Gen3 x8, or 2 x Gen3 x8, 1 x Gen3 x16)
* Phytium S2500 (1 x Gen3 x1, 1 x Gen3 x16 or 2 x Gen3 x8)
* Qualcomm Snapdragon 810 (2 x Gen2 x1)
* Qualcomm Snapdragon 820 (3 x Gen2 x1)
* Qualcomm Snapdragon 835 (1 x Gen2 x1)
* Qualcomm Snapdragon 845 (1 x Gen2 x1, 1 x Gen3 x1)
* RealTek RTD1294 (1 x Gen1 x1)
* RealTek RTD1295/RTD1296 (1 x Gen1 x1, 1 x Gen2 x1)
* RealTek RTD1395 (1 x Gen2 x1)
* RealTek RTD1619 (2 x Gen2 x1)
* Rockchip RK3399 (1 x Gen2 x4)
* Rockchip RK3566 (1 x Gen2 x1, multiplexed with SATA)
* Rockchip RK3568 (1 x Gen3 x2 or 2 x Gen3 x1, 1 x Gen2 x1, [multiplexed with SATA/QSGMII](https://www.cnx-software.com/2020/12/16/rockchip-rk3566-and-rk3568-datasheets-and-features-comparison/))
* Rockchip RK3588 (4 Gen3 lanes: x4, x2+x2, x2+x1+x1, x1+x1+x1+x1, 3 x Gen2 x1, [multiplexed with SATA/USB3](https://www.cnx-software.com/2021/12/16/rockchip-rk3588-datasheet-sbc-coming-soon/))
* Rockchip RK3588S (2 x Gen2 x1, [multiplexed with SATA/USB3](https://www.cnx-software.com/2022/01/12/rockchip-rk3588s-cost-optimized-cortex-a76-a55-processor/))
* Samsung Exynos 9810 (1 x Gen3 x1, 1 x Gen2 x1)
* Texas Instruments AM64x (1 x Gen2 x1)
* Texas Instruments AM654 (2 x Gen3 x1)
* Texas Instruments J721E/AM752x (up to 4 x Gen3 x2)

## Server SoCs

* AMD A1100 (8 Gen3 lanes)
* Ampere Altra / Altra Max (up to 128 Gen4 lanes)
* AnnapurnaLabs (Amazon) Alpine AL-324
* APM X-Gene 1 (17 PCIe Gen3 lanes)
* APM X-Gene 2 (17 PCIe Gen3 lanes)
* APM X-Gene 3 (42 PCIe Gen3 lanes)
* Broadcom Vulcan AKA ThunderX2 (up to 56 PCIe Gen3 lanes with 14 controllers. Supported widths x1, x2, x4, x8 and x16)
* Cavium ThunderX (multiple x4/x8 Gen3 ports)
* HiSilicon Hi1610/Hi1612 (16 Gen3 lanes)
* HiSilicon Hi1616 (46 Gen3 lanes)
* HiSilicon Hi1620 AKA Kunpeng 920 (40 Gen4 lanes including 16 that can be used for CCIX --> Cache Coherent Interconnect for Accelerators)
* Marvell Armada 37x0 (1 x Gen2 x1)
* Marvell Armada 70x0 (1 x Gen3 x4, 2 x Gen3 x1)
* Marvell Armada 80x0 (1 x Gen3 x4, 1 x Gen3 x2, 4 x Gen3 x1)
* Qualcomm Centriq 2400 (32 Gen3 lanes up to x16, 6 controllers)
* Socionext SC2A11 (4 Gen2 lanes)

## Network/router SoCs

Pointless list since majority of network processors today is ARM based with excellent PCIe capabilities and 'offload engines' (buy a 40GbE PCIe network card for your Xeon box and you most probably end up with a quad or octa core ARMv8 CPU on this card)

* Broadcom BCM5871x Series (8 Gen2 lanes)
* Broadcom BCM5880x Series (16 Gen3 lanes)
* NXP QorIQ Layerscape (from 1 x Gen2 x1 up to 24 Gen4 lanes, supporting ports as wide as x8)
