# ARMv8 SoCs with PCIe support

## Multi purpose SoCs

* Allwinner H6 (1 x Gen2 x1). Note: [quirky controller not able to work with -- non existing -- generic drivers](https://linux-sunxi.org/Mainlining_Effort#cite_note-h6-pcie-4)
* Amlogic S922X/A311D (1 x Gen2 x1)
* MediaTek MT7622 (2 x Gen2 x1)
* NXP i.MX8 (up to 1 x Gen3 x2 or 2 x Gen3 x1, another x1 multiplexed with SATA)
* Qualcomm Snapdragon 810 (2 x Gen2 x1)
* Qualcomm Snapdragon 820 (3 x Gen2 x1)
* Qualcomm Snapdragon 835 (1 x Gen2 x1)
* Qualcomm Snapdragon 845 (1 x Gen2 x1, 1 x Gen3 x1)
* RealTek RTD1295/RTD1296 (1 x Gen1 x1, 1 x Gen2 x1)
* Rockchip RK3399 (1 x Gen2 x4)
* Samsung Exynos 9810 (1 x Gen3 x1, 1 x Gen2 x1)
* Texas Instruments AM654 (2 x Gen3 x1)

## Server SoCs

* AMD A1100 (8 Gen3 lanes)
* AnnapurnaLabs (Amazon) Alpine AL-324
* APM X-Gene 1 (17 PCIe Gen 3 lanes)
* APM X-Gene 2 (17 PCIe Gen 3 lanes)
* APM X-Gene 3 (42 PCIe Gen 3 lanes)
* Broadcom Vulcan AKA ThunderX2 (up to 56 PCIe Gen3 lanes with 14 controllers. Supported widths x1, x2, x4, x8 and x16)
* Cavium ThunderX (multiple x4/x8 Gen3 ports)
* HiSilicon Hi1610/Hi1612 (16 Gen3 lanes)
* HiSilicon Hi1616 (46 Gen3 lanes)
* HiSilicon Hi1620 (40 Gen4 lanes)
* Huawei Kunpeng 920 (40 Gen4 lanes including 16 that can be used for CCIX --> Cache Coherent Interconnect for Accelerators)
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