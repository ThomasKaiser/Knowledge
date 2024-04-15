# Quick Preview of ROCK 5 ITX

![I/O ports view](../media/rock5-itx-1.jpg)

**(Work in progress)**

<details>
  <summary>More pictures</summary>

  ![Top view](../media/rock5-itx-2.jpg)
  
  ![Connector side view](../media/rock5-itx-3.jpg)

  ![SATA side view](../media/rock5-itx-4.jpg)

  ![Board components](../media/rock5-itx-6.jpg)

  ![SD card, MIPI, Maskrom button, touchpad, eDP](../media/rock5-itx-7.jpg)

  ![M.2 key E, front panel header, audio jacks, RA620-1, ES8316](../media/rock5-itx-8.jpg)  

  ![Board with I/O Shield](../media/rock5-itx-9.jpg)

  ![Cardboard package](../media/rock5-itx-10.jpg)

</details>

## Overview

April 2024 Radxa started to send out developer samples of their RK3588 based [Mini-ITX](https://en.wikipedia.org/wiki/Mini-ITX) v1.11 board (for RK3588 basics and some notes about software support status see [Rock 5B](Quick_Preview_of_ROCK_5B.md)). Official documentation will once appear [here](https://docs.radxa.com/en/rock5/rock5itx) and for now you can see a block diagram [there](https://docs.radxa.com/en/assets/images/rock5itx-interface-overview-1266d3c0b4e745372a48a473d78c3cdc.webp)

The board measures 170x170mm in size as it follows the Mini-ITX standard with externally accessible connectors all on the 'back side' accompanied by an appropriate I/O shield. From left to right there's

  * 5.5/2.1 mm centre positive DC-IN: wide input range (voltage range not tested/confirmed yet) but [**important** to use 12V when powering 5.25" HDDs by the board](#sata-power-ports)
  * USB-C with OTG (USB 3.0 / FullSpeed / 5Gbps) and DisplayPort, no powering through this port
  * HDMI IN
  * RJ45 / 2.5GbE provided by RTL8125BG + USB 2.0 behind 4-port Terminus Inc. USB2 hub
  * another RJ45 / 2.5GbE provided by RTL8125BG + USB 2.0 behind same USB2 hub
  * 2 x USB3 5Gbps behind [Genesys Logic GL3523 USB3 hub](https://linux-hardware.org/?id=usb:05e3-0620) and HDMI out (8K capable)
  * 2 x USB3 5Gbps also behind the same GL3523 hub and HDMI out (4K capable, behind a Rockchip RK620-1 / Radxa RA620-1 DisplayPort-to-HDMI converter chip)
  * headphone/mic (via I2S attached [ES8316](http://everest-semi.com/pdf/ES8316%20PB.pdf))
  * [S/PDIF](https://en.wikipedia.org/wiki/S/PDIF)

On the next board side we find the following internal connectors:

  * front panel header (HDD LED, power LED, reset and power button)
  * audio header
  * USB2 header for front panel USB receptacles (two ports also behind same USB2 hub)
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
  * Maskrom button

## I/O capabilities

Some protocols on RK3588 are pinmuxed so the board designer has to decide between PCIe, USB3 and SATA in some cases. For a pinmuxing overview [see here](https://www.cnx-software.com/2021/12/16/rockchip-rk3588-datasheet-sbc-coming-soon/) and for a general RK3588 PCIe overview [see there](https://github.com/ThomasKaiser/Knowledge/blob/master/articles/Quick_Preview_of_ROCK_5B.md#pcie). On this board the seven available PCIe lanes are used as follows:

  * two of the Gen3 lanes are routed to the M.2 key M slot to be used with a SSD or M.2 storage/network adapters
  * the other two Gen3 lanes connect to the [ASM1164 SATA host controller](https://www.asmedia.com.tw/product/17fYQ85SPeqG8MT8/58dYQ8bxZ4UR9wG5) that provides the 4 SATA 6Gbps ports on the board. Talking about bandwidth PCIe Gen3 x2 and 4 x SATA at 6.0 Gbit/s with 8b/10b coding are a pretty close match as such not only four pieces of spinning rust but also fast SATA SSDs should be able to be accessed at full speed in parallel
  * two Gen2 lanes are used to attach the RTL8125BG Ethernet controllers
  * the last Gen2 lane is routed to the M.2 key E slot to be used with PCIe peripherals like a Wi-Fi card or when switched to SATA mode as another (and this time SoC native) SATA port. Making use of SATA requires a passive adapter like Radxa's 'M.2 E Key to SATA Adapter' that unfortunately seems to be sold out everywhere
  * all four USB3 receptacles are behind a GL3523 USB3 hub and as such have to share bandwidth
  * the same goes for the four USB2 ports (two receptables and two on headers) that are behind an USB2 hub
  * The USB-C receptacle defaults to USB3 host mode (bandwidth *not* shared with the other USB3 ports) but can also be switched into OTG mode via a device-tree overlay to access the device via ADB or as an USB gadget.
  * the TF card slot is UHS-I/SDR104 capable
  * the eMMC interface utilizes HS400 mode

<details>
  <summary>lspci -vv</summary>

    root@rock-5-itx:~# lspci -vv
    0001:10:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd RK3588 (rev 01) (prog-if 00 [Normal decode])
    	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx+
    	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
    	Latency: 0
    	Interrupt: pin A routed to IRQ 152
    	Bus: primary=10, secondary=11, subordinate=11, sec-latency=0
    	I/O behind bridge: [disabled]
    	Memory behind bridge: f1200000-f12fffff [size=1M]
    	Prefetchable memory behind bridge: [disabled]
    	Secondary status: 66MHz- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- <SERR- <PERR-
    	Expansion ROM at f1300000 [virtual] [disabled] [size=64K]
    	BridgeCtl: Parity- SERR+ NoISA- VGA- VGA16- MAbort- >Reset- FastB2B-
    		PriDiscTmr- SecDiscTmr- DiscTmrStat- DiscTmrSERREn-
    	Capabilities: [40] Power Management version 3
    		Flags: PMEClk- DSI- D1+ D2+ AuxCurrent=375mA PME(D0+,D1+,D2-,D3hot+,D3cold-)
    		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
    	Capabilities: [50] MSI: Enable+ Count=16/32 Maskable- 64bit+
    		Address: 00000000fe670040  Data: 0000
    	Capabilities: [70] Express (v2) Root Port (Slot-), MSI 08
    		DevCap:	MaxPayload 256 bytes, PhantFunc 0
    			ExtTag- RBE+
    		DevCtl:	CorrErr- NonFatalErr- FatalErr- UnsupReq-
    			RlxdOrd+ ExtTag- PhantFunc- AuxPwr- NoSnoop-
    			MaxPayload 128 bytes, MaxReadReq 512 bytes
    		DevSta:	CorrErr+ NonFatalErr- FatalErr- UnsupReq- AuxPwr- TransPend-
    		LnkCap:	Port #0, Speed 8GT/s, Width x2, ASPM L1, Exit Latency L1 <16us
    			ClockPM- Surprise- LLActRep+ BwNot+ ASPMOptComp+
    		LnkCtl:	ASPM Disabled; RCB 64 bytes, Disabled- CommClk+
    			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
    		LnkSta:	Speed 8GT/s (ok), Width x2 (ok)
    			TrErr- Train- SlotClk+ DLActive+ BWMgmt+ ABWMgmt+
    		RootCap: CRSVisible+
    		RootCtl: ErrCorrectable- ErrNon-Fatal- ErrFatal- PMEIntEna+ CRSVisible+
    		RootSta: PME ReqID 0000, PMEStatus- PMEPending-
    		DevCap2: Completion Timeout: Not Supported, TimeoutDis+ NROPrPrP+ LTR+
    			 10BitTagComp- 10BitTagReq- OBFF Via message/WAKE#, ExtFmt+ EETLPPrefix+, MaxEETLPPrefixes 1
    			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
    			 FRS- LN System CLS Not Supported, TPHComp- ExtTPHComp- ARIFwd+
    			 AtomicOpsCap: Routing- 32bit- 64bit- 128bitCAS-
    		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis- LTR+ OBFF Disabled, ARIFwd-
    			 AtomicOpsCtl: ReqEn- EgressBlck-
    		LnkCap2: Supported Link Speeds: 2.5-8GT/s, Crosslink- Retimer- 2Retimers- DRS-
    		LnkCtl2: Target Link Speed: 8GT/s, EnterCompliance- SpeedDis-
    			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
    			 Compliance De-emphasis: -6dB
    		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete+ EqualizationPhase1+
    			 EqualizationPhase2+ EqualizationPhase3+ LinkEqualizationRequest-
    			 Retimer- 2Retimers- CrosslinkRes: unsupported
    	Capabilities: [b0] MSI-X: Enable- Count=128 Masked-
    		Vector table: BAR=4 offset=00020000
    		PBA: BAR=4 offset=00028000
    	Capabilities: [100 v2] Advanced Error Reporting
    		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    		CESta:	RxErr+ BadTLP- BadDLLP+ Rollover- Timeout+ AdvNonFatalErr-
    		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
    		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
    			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
    		HeaderLog: 00000000 00000000 00000000 00000000
    		RootCmd: CERptEn- NFERptEn- FERptEn-
    		RootSta: CERcvd- MultCERcvd- UERcvd- MultUERcvd-
    			 FirstFatal- NonFatalMsg- FatalMsg- IntMsg 9
    		ErrorSrc: ERR_COR: 0000 ERR_FATAL/NONFATAL: 0000
    	Capabilities: [148 v1] Secondary PCI Express
    		LnkCtl3: LnkEquIntrruptEn- PerformEqu-
    		LaneErrStat: LaneErr at lane: 1
    	Capabilities: [180 v1] L1 PM Substates
    		L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2- ASPM_L1.1- L1_PM_Substates+
    			  PortCommonModeRestoreTime=10us PortTPowerOnTime=10us
    		L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
    			   T_CommonMode=10us
    		L1SubCtl2: T_PwrOn=10us
    	Capabilities: [190 v1] Vendor Specific Information: ID=0002 Rev=4 Len=100 <?>
    	Kernel driver in use: pcieport
    
    0001:11:00.0 SATA controller: ASMedia Technology Inc. ASM1164 Serial ATA AHCI Controller (rev 02) (prog-if 01 [AHCI 1.0])
    	Subsystem: ASMedia Technology Inc. Device 2116
    	Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
    	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
    	Latency: 0
    	Interrupt: pin A routed to IRQ 161
    	Region 0: Memory at f1280000 (32-bit, non-prefetchable) [size=8K]
    	Region 5: Memory at f1282000 (32-bit, non-prefetchable) [size=8K]
    	Expansion ROM at f1200000 [virtual] [disabled] [size=512K]
    	Capabilities: [40] Power Management version 3
    		Flags: PMEClk- DSI+ D1- D2- AuxCurrent=0mA PME(D0+,D1-,D2-,D3hot+,D3cold+)
    		Status: D0 NoSoftRst- PME-Enable- DSel=0 DScale=0 PME-
    	Capabilities: [50] MSI: Enable+ Count=1/1 Maskable- 64bit+
    		Address: 00000000fe670040  Data: 0000
    	Capabilities: [80] Express (v2) Endpoint, MSI 00
    		DevCap:	MaxPayload 256 bytes, PhantFunc 0, Latency L0s unlimited, L1 unlimited
    			ExtTag+ AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 0.000W
    		DevCtl:	CorrErr- NonFatalErr- FatalErr- UnsupReq-
    			RlxdOrd+ ExtTag+ PhantFunc- AuxPwr- NoSnoop+
    			MaxPayload 128 bytes, MaxReadReq 256 bytes
    		DevSta:	CorrErr- NonFatalErr- FatalErr- UnsupReq- AuxPwr- TransPend-
    		LnkCap:	Port #0, Speed 8GT/s, Width x2, ASPM L1, Exit Latency L1 <64us
    			ClockPM+ Surprise- LLActRep- BwNot- ASPMOptComp+
    		LnkCtl:	ASPM Disabled; RCB 64 bytes, Disabled- CommClk+
    			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
    		LnkSta:	Speed 8GT/s (ok), Width x2 (ok)
    			TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
    		DevCap2: Completion Timeout: Not Supported, TimeoutDis- NROPrPrP- LTR-
    			 10BitTagComp- 10BitTagReq- OBFF Not Supported, ExtFmt- EETLPPrefix-
    			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
    			 FRS- TPHComp- ExtTPHComp-
    			 AtomicOpsCap: 32bit- 64bit- 128bitCAS-
    		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis- LTR- OBFF Disabled,
    			 AtomicOpsCtl: ReqEn-
    		LnkCap2: Supported Link Speeds: 2.5-8GT/s, Crosslink- Retimer- 2Retimers- DRS-
    		LnkCtl2: Target Link Speed: 8GT/s, EnterCompliance- SpeedDis+
    			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
    			 Compliance De-emphasis: -6dB
    		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete+ EqualizationPhase1+
    			 EqualizationPhase2+ EqualizationPhase3+ LinkEqualizationRequest-
    			 Retimer- 2Retimers- CrosslinkRes: unsupported
    	Capabilities: [100 v1] Advanced Error Reporting
    		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
    		AERCap:	First Error Pointer: 00, ECRCGenCap- ECRCGenEn- ECRCChkCap- ECRCChkEn-
    			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
    		HeaderLog: 00000000 00000000 00000000 00000000
    	Capabilities: [130 v1] Secondary PCI Express
    		LnkCtl3: LnkEquIntrruptEn- PerformEqu-
    		LaneErrStat: 0
    	Kernel driver in use: ahci
    
    0003:30:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd RK3588 (rev 01) (prog-if 00 [Normal decode])
    	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx+
    	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
    	Latency: 0
    	Interrupt: pin A routed to IRQ 140
    	Bus: primary=30, secondary=31, subordinate=31, sec-latency=0
    	I/O behind bridge: 00001000-00001fff [size=4K]
    	Memory behind bridge: f3200000-f32fffff [size=1M]
    	Prefetchable memory behind bridge: [disabled]
    	Secondary status: 66MHz- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- <SERR- <PERR-
    	Expansion ROM at f3300000 [virtual] [disabled] [size=64K]
    	BridgeCtl: Parity- SERR+ NoISA- VGA- VGA16- MAbort- >Reset- FastB2B-
    		PriDiscTmr- SecDiscTmr- DiscTmrStat- DiscTmrSERREn-
    	Capabilities: [40] Power Management version 3
    		Flags: PMEClk- DSI- D1+ D2+ AuxCurrent=375mA PME(D0+,D1+,D2-,D3hot+,D3cold-)
    		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
    	Capabilities: [50] MSI: Enable+ Count=16/32 Maskable- 64bit+
    		Address: 00000000fe650040  Data: 0000
    	Capabilities: [70] Express (v2) Root Port (Slot-), MSI 08
    		DevCap:	MaxPayload 256 bytes, PhantFunc 0
    			ExtTag- RBE+
    		DevCtl:	CorrErr- NonFatalErr- FatalErr- UnsupReq-
    			RlxdOrd+ ExtTag- PhantFunc- AuxPwr- NoSnoop-
    			MaxPayload 128 bytes, MaxReadReq 512 bytes
    		DevSta:	CorrErr- NonFatalErr- FatalErr- UnsupReq- AuxPwr- TransPend-
    		LnkCap:	Port #0, Speed 5GT/s, Width x1, ASPM L1, Exit Latency L1 <16us
    			ClockPM- Surprise- LLActRep+ BwNot+ ASPMOptComp+
    		LnkCtl:	ASPM Disabled; RCB 64 bytes, Disabled- CommClk+
    			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
    		LnkSta:	Speed 5GT/s (ok), Width x1 (ok)
    			TrErr- Train- SlotClk+ DLActive+ BWMgmt+ ABWMgmt+
    		RootCap: CRSVisible+
    		RootCtl: ErrCorrectable- ErrNon-Fatal- ErrFatal- PMEIntEna+ CRSVisible+
    		RootSta: PME ReqID 0000, PMEStatus- PMEPending-
    		DevCap2: Completion Timeout: Not Supported, TimeoutDis+ NROPrPrP+ LTR+
    			 10BitTagComp- 10BitTagReq- OBFF Via message/WAKE#, ExtFmt+ EETLPPrefix+, MaxEETLPPrefixes 1
    			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
    			 FRS- LN System CLS Not Supported, TPHComp- ExtTPHComp- ARIFwd+
    			 AtomicOpsCap: Routing- 32bit- 64bit- 128bitCAS-
    		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis- LTR+ OBFF Disabled, ARIFwd-
    			 AtomicOpsCtl: ReqEn- EgressBlck-
    		LnkCap2: Supported Link Speeds: 2.5-5GT/s, Crosslink- Retimer- 2Retimers- DRS-
    		LnkCtl2: Target Link Speed: 5GT/s, EnterCompliance- SpeedDis-
    			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
    			 Compliance De-emphasis: -6dB
    		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete- EqualizationPhase1-
    			 EqualizationPhase2- EqualizationPhase3- LinkEqualizationRequest-
    			 Retimer- 2Retimers- CrosslinkRes: unsupported
    	Capabilities: [b0] MSI-X: Enable- Count=128 Masked-
    		Vector table: BAR=4 offset=00020000
    		PBA: BAR=4 offset=00028000
    	Capabilities: [100 v2] Advanced Error Reporting
    		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
    		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
    			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
    		HeaderLog: 00000000 00000000 00000000 00000000
    		RootCmd: CERptEn- NFERptEn- FERptEn-
    		RootSta: CERcvd- MultCERcvd- UERcvd- MultUERcvd-
    			 FirstFatal- NonFatalMsg- FatalMsg- IntMsg 9
    		ErrorSrc: ERR_COR: 0000 ERR_FATAL/NONFATAL: 0000
    	Capabilities: [148 v1] Secondary PCI Express
    		LnkCtl3: LnkEquIntrruptEn- PerformEqu-
    		LaneErrStat: 0
    	Capabilities: [180 v1] L1 PM Substates
    		L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2- ASPM_L1.1- L1_PM_Substates+
    			  PortCommonModeRestoreTime=10us PortTPowerOnTime=10us
    		L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
    			   T_CommonMode=10us
    		L1SubCtl2: T_PwrOn=10us
    	Capabilities: [190 v1] Vendor Specific Information: ID=0002 Rev=4 Len=100 <?>
    	Kernel driver in use: pcieport
    
    0003:31:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
    	Subsystem: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller
    	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
    	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
    	Latency: 0, Cache Line Size: 64 bytes
    	Interrupt: pin A routed to IRQ 139
    	Region 0: I/O ports at 1000 [size=256]
    	Region 2: Memory at f3200000 (64-bit, non-prefetchable) [size=64K]
    	Region 4: Memory at f3210000 (64-bit, non-prefetchable) [size=16K]
    	Capabilities: [40] Power Management version 3
    		Flags: PMEClk- DSI- D1+ D2+ AuxCurrent=375mA PME(D0+,D1+,D2+,D3hot+,D3cold+)
    		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
    	Capabilities: [50] MSI: Enable- Count=1/1 Maskable+ 64bit+
    		Address: 0000000000000000  Data: 0000
    		Masking: 00000000  Pending: 00000000
    	Capabilities: [70] Express (v2) Endpoint, MSI 01
    		DevCap:	MaxPayload 256 bytes, PhantFunc 0, Latency L0s <512ns, L1 <64us
    			ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 0.000W
    		DevCtl:	CorrErr- NonFatalErr- FatalErr- UnsupReq-
    			RlxdOrd+ ExtTag- PhantFunc- AuxPwr- NoSnoop-
    			MaxPayload 128 bytes, MaxReadReq 4096 bytes
    		DevSta:	CorrErr- NonFatalErr- FatalErr- UnsupReq- AuxPwr+ TransPend-
    		LnkCap:	Port #0, Speed 5GT/s, Width x1, ASPM L0s L1, Exit Latency L0s unlimited, L1 <64us
    			ClockPM+ Surprise- LLActRep- BwNot- ASPMOptComp+
    		LnkCtl:	ASPM Disabled; RCB 64 bytes, Disabled- CommClk+
    			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
    		LnkSta:	Speed 5GT/s (ok), Width x1 (ok)
    			TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
    		DevCap2: Completion Timeout: Range ABCD, TimeoutDis+ NROPrPrP- LTR+
    			 10BitTagComp- 10BitTagReq- OBFF Via message/WAKE#, ExtFmt- EETLPPrefix-
    			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
    			 FRS- TPHComp+ ExtTPHComp-
    			 AtomicOpsCap: 32bit- 64bit- 128bitCAS-
    		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis- LTR+ OBFF Disabled,
    			 AtomicOpsCtl: ReqEn-
    		LnkCap2: Supported Link Speeds: 2.5-5GT/s, Crosslink- Retimer- 2Retimers- DRS-
    		LnkCtl2: Target Link Speed: 5GT/s, EnterCompliance- SpeedDis-
    			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
    			 Compliance De-emphasis: -6dB
    		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete- EqualizationPhase1-
    			 EqualizationPhase2- EqualizationPhase3- LinkEqualizationRequest-
    			 Retimer- 2Retimers- CrosslinkRes: unsupported
    	Capabilities: [b0] MSI-X: Enable+ Count=32 Masked-
    		Vector table: BAR=4 offset=00000000
    		PBA: BAR=4 offset=00000800
    	Capabilities: [d0] Vital Product Data
    pcilib: sysfs_read_vpd: read failed: Input/output error
    		Not readable
    	Capabilities: [100 v2] Advanced Error Reporting
    		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
    		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
    		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
    			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
    		HeaderLog: 00000000 00000000 00000000 00000000
    	Capabilities: [148 v1] Virtual Channel
    		Caps:	LPEVC=0 RefClk=100ns PATEntryBits=1
    		Arb:	Fixed- WRR32- WRR64- WRR128-
    		Ctrl:	ArbSelect=Fixed
    		Status:	InProgress-
    		VC0:	Caps:	PATOffset=00 MaxTimeSlots=1 RejSnoopTrans-
    			Arb:	Fixed- WRR32- WRR64- WRR128- TWRR128- WRR256-
    			Ctrl:	Enable+ ID=0 ArbSelect=Fixed TC/VC=ff
    			Status:	NegoPending- InProgress-
    	Capabilities: [168 v1] Device Serial Number 02-00-00-00-68-4c-e0-00
    	Capabilities: [178 v1] Transaction Processing Hints
    		No steering table available
    	Capabilities: [204 v1] Latency Tolerance Reporting
    		Max snoop latency: 0ns
    		Max no snoop latency: 0ns
    	Capabilities: [20c v1] L1 PM Substates
    		L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+ L1_PM_Substates+
    			  PortCommonModeRestoreTime=150us PortTPowerOnTime=150us
    		L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
    			   T_CommonMode=0us LTR1.2_Threshold=0ns
    		L1SubCtl2: T_PwrOn=10us
    	Capabilities: [21c v1] Vendor Specific Information: ID=0002 Rev=4 Len=100 <?>
    	Kernel driver in use: r8125
    	Kernel modules: r8125
    
    0004:40:00.0 PCI bridge: Fuzhou Rockchip Electronics Co., Ltd RK3588 (rev 01) (prog-if 00 [Normal decode])
    	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR+ FastB2B- DisINTx+
    	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
    	Latency: 0
    	Interrupt: pin A routed to IRQ 163
    	Bus: primary=40, secondary=41, subordinate=41, sec-latency=0
    	I/O behind bridge: 00000000-00000fff [size=4K]
    	Memory behind bridge: f4200000-f42fffff [size=1M]
    	Prefetchable memory behind bridge: [disabled]
    	Secondary status: 66MHz- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- <SERR- <PERR-
    	Expansion ROM at f4300000 [virtual] [disabled] [size=64K]
    	BridgeCtl: Parity- SERR+ NoISA- VGA- VGA16- MAbort- >Reset- FastB2B-
    		PriDiscTmr- SecDiscTmr- DiscTmrStat- DiscTmrSERREn-
    	Capabilities: [40] Power Management version 3
    		Flags: PMEClk- DSI- D1+ D2+ AuxCurrent=375mA PME(D0+,D1+,D2-,D3hot+,D3cold-)
    		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
    	Capabilities: [50] MSI: Enable+ Count=16/32 Maskable- 64bit+
    		Address: 00000000fe650040  Data: 0000
    	Capabilities: [70] Express (v2) Root Port (Slot-), MSI 08
    		DevCap:	MaxPayload 256 bytes, PhantFunc 0
    			ExtTag- RBE+
    		DevCtl:	CorrErr- NonFatalErr- FatalErr- UnsupReq-
    			RlxdOrd+ ExtTag- PhantFunc- AuxPwr- NoSnoop-
    			MaxPayload 128 bytes, MaxReadReq 512 bytes
    		DevSta:	CorrErr- NonFatalErr- FatalErr- UnsupReq- AuxPwr- TransPend-
    		LnkCap:	Port #0, Speed 5GT/s, Width x1, ASPM L1, Exit Latency L1 <16us
    			ClockPM- Surprise- LLActRep+ BwNot+ ASPMOptComp+
    		LnkCtl:	ASPM Disabled; RCB 64 bytes, Disabled- CommClk+
    			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
    		LnkSta:	Speed 5GT/s (ok), Width x1 (ok)
    			TrErr- Train- SlotClk+ DLActive+ BWMgmt+ ABWMgmt+
    		RootCap: CRSVisible+
    		RootCtl: ErrCorrectable- ErrNon-Fatal- ErrFatal- PMEIntEna+ CRSVisible+
    		RootSta: PME ReqID 0000, PMEStatus- PMEPending-
    		DevCap2: Completion Timeout: Not Supported, TimeoutDis+ NROPrPrP+ LTR+
    			 10BitTagComp- 10BitTagReq- OBFF Via message/WAKE#, ExtFmt+ EETLPPrefix+, MaxEETLPPrefixes 1
    			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
    			 FRS- LN System CLS Not Supported, TPHComp- ExtTPHComp- ARIFwd+
    			 AtomicOpsCap: Routing- 32bit- 64bit- 128bitCAS-
    		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis- LTR+ OBFF Disabled, ARIFwd-
    			 AtomicOpsCtl: ReqEn- EgressBlck-
    		LnkCap2: Supported Link Speeds: 2.5-5GT/s, Crosslink- Retimer- 2Retimers- DRS-
    		LnkCtl2: Target Link Speed: 5GT/s, EnterCompliance- SpeedDis-
    			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
    			 Compliance De-emphasis: -6dB
    		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete- EqualizationPhase1-
    			 EqualizationPhase2- EqualizationPhase3- LinkEqualizationRequest-
    			 Retimer- 2Retimers- CrosslinkRes: unsupported
    	Capabilities: [b0] MSI-X: Enable- Count=128 Masked-
    		Vector table: BAR=4 offset=00020000
    		PBA: BAR=4 offset=00028000
    	Capabilities: [100 v2] Advanced Error Reporting
    		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
    		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
    			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
    		HeaderLog: 00000000 00000000 00000000 00000000
    		RootCmd: CERptEn- NFERptEn- FERptEn-
    		RootSta: CERcvd- MultCERcvd- UERcvd- MultUERcvd-
    			 FirstFatal- NonFatalMsg- FatalMsg- IntMsg 9
    		ErrorSrc: ERR_COR: 0000 ERR_FATAL/NONFATAL: 0000
    	Capabilities: [148 v1] Secondary PCI Express
    		LnkCtl3: LnkEquIntrruptEn- PerformEqu-
    		LaneErrStat: 0
    	Capabilities: [180 v1] L1 PM Substates
    		L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2- ASPM_L1.1- L1_PM_Substates+
    			  PortCommonModeRestoreTime=10us PortTPowerOnTime=10us
    		L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
    			   T_CommonMode=10us
    		L1SubCtl2: T_PwrOn=10us
    	Capabilities: [190 v1] Vendor Specific Information: ID=0002 Rev=4 Len=100 <?>
    	Kernel driver in use: pcieport
    
    0004:41:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
    	Subsystem: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller
    	Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
    	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
    	Latency: 0, Cache Line Size: 64 bytes
    	Interrupt: pin A routed to IRQ 162
    	Region 0: I/O ports at 400000 [size=256]
    	Region 2: Memory at f4200000 (64-bit, non-prefetchable) [size=64K]
    	Region 4: Memory at f4210000 (64-bit, non-prefetchable) [size=16K]
    	Capabilities: [40] Power Management version 3
    		Flags: PMEClk- DSI- D1+ D2+ AuxCurrent=375mA PME(D0+,D1+,D2+,D3hot+,D3cold+)
    		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
    	Capabilities: [50] MSI: Enable- Count=1/1 Maskable+ 64bit+
    		Address: 0000000000000000  Data: 0000
    		Masking: 00000000  Pending: 00000000
    	Capabilities: [70] Express (v2) Endpoint, MSI 01
    		DevCap:	MaxPayload 256 bytes, PhantFunc 0, Latency L0s <512ns, L1 <64us
    			ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 0.000W
    		DevCtl:	CorrErr- NonFatalErr- FatalErr- UnsupReq-
    			RlxdOrd+ ExtTag- PhantFunc- AuxPwr- NoSnoop-
    			MaxPayload 128 bytes, MaxReadReq 4096 bytes
    		DevSta:	CorrErr- NonFatalErr- FatalErr- UnsupReq- AuxPwr+ TransPend-
    		LnkCap:	Port #0, Speed 5GT/s, Width x1, ASPM L0s L1, Exit Latency L0s unlimited, L1 <64us
    			ClockPM+ Surprise- LLActRep- BwNot- ASPMOptComp+
    		LnkCtl:	ASPM Disabled; RCB 64 bytes, Disabled- CommClk+
    			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
    		LnkSta:	Speed 5GT/s (ok), Width x1 (ok)
    			TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
    		DevCap2: Completion Timeout: Range ABCD, TimeoutDis+ NROPrPrP- LTR+
    			 10BitTagComp- 10BitTagReq- OBFF Via message/WAKE#, ExtFmt- EETLPPrefix-
    			 EmergencyPowerReduction Not Supported, EmergencyPowerReductionInit-
    			 FRS- TPHComp+ ExtTPHComp-
    			 AtomicOpsCap: 32bit- 64bit- 128bitCAS-
    		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis- LTR+ OBFF Disabled,
    			 AtomicOpsCtl: ReqEn-
    		LnkCap2: Supported Link Speeds: 2.5-5GT/s, Crosslink- Retimer- 2Retimers- DRS-
    		LnkCtl2: Target Link Speed: 5GT/s, EnterCompliance- SpeedDis-
    			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
    			 Compliance De-emphasis: -6dB
    		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete- EqualizationPhase1-
    			 EqualizationPhase2- EqualizationPhase3- LinkEqualizationRequest-
    			 Retimer- 2Retimers- CrosslinkRes: unsupported
    	Capabilities: [b0] MSI-X: Enable+ Count=32 Masked-
    		Vector table: BAR=4 offset=00000000
    		PBA: BAR=4 offset=00000800
    	Capabilities: [d0] Vital Product Data
    pcilib: sysfs_read_vpd: read failed: Input/output error
    		Not readable
    	Capabilities: [100 v2] Advanced Error Reporting
    		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
    		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq+ ACSViol-
    		UESvrt:	DLP+ SDES+ TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
    		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr-
    		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- AdvNonFatalErr+
    		AERCap:	First Error Pointer: 00, ECRCGenCap+ ECRCGenEn- ECRCChkCap+ ECRCChkEn-
    			MultHdrRecCap- MultHdrRecEn- TLPPfxPres- HdrLogCap-
    		HeaderLog: 00000000 00000000 00000000 00000000
    	Capabilities: [148 v1] Virtual Channel
    		Caps:	LPEVC=0 RefClk=100ns PATEntryBits=1
    		Arb:	Fixed- WRR32- WRR64- WRR128-
    		Ctrl:	ArbSelect=Fixed
    		Status:	InProgress-
    		VC0:	Caps:	PATOffset=00 MaxTimeSlots=1 RejSnoopTrans-
    			Arb:	Fixed- WRR32- WRR64- WRR128- TWRR128- WRR256-
    			Ctrl:	Enable+ ID=0 ArbSelect=Fixed TC/VC=ff
    			Status:	NegoPending- InProgress-
    	Capabilities: [168 v1] Device Serial Number 02-00-00-00-68-4c-e0-00
    	Capabilities: [178 v1] Transaction Processing Hints
    		No steering table available
    	Capabilities: [204 v1] Latency Tolerance Reporting
    		Max snoop latency: 0ns
    		Max no snoop latency: 0ns
    	Capabilities: [20c v1] L1 PM Substates
    		L1SubCap: PCI-PM_L1.2+ PCI-PM_L1.1+ ASPM_L1.2+ ASPM_L1.1+ L1_PM_Substates+
    			  PortCommonModeRestoreTime=150us PortTPowerOnTime=150us
    		L1SubCtl1: PCI-PM_L1.2- PCI-PM_L1.1- ASPM_L1.2- ASPM_L1.1-
    			   T_CommonMode=0us LTR1.2_Threshold=0ns
    		L1SubCtl2: T_PwrOn=10us
    	Capabilities: [21c v1] Vendor Specific Information: ID=0002 Rev=4 Len=100 <?>
    	Kernel driver in use: r8125
    	Kernel modules: r8125

</details>

<details>
  <summary>lsusb with all seven USB receptacles occupied by storage devices</summary>

    root@rock-5-itx:~# lsusb
    Bus 008 Device 002: ID 2109:0715 VIA Labs, Inc. VL817 SATA Adaptor
    Bus 008 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 006 Device 003: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
    Bus 006 Device 007: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
    Bus 006 Device 006: ID 152d:3562 JMicron Technology Corp. / JMicron USA Technology Corp. JMS567 SATA 6Gb/s bridge
    Bus 006 Device 005: ID 0781:5583 SanDisk Corp. Ultra Fit
    Bus 006 Device 002: ID 05e3:0620 Genesys Logic, Inc. USB3.2 Hub
    Bus 006 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 005 Device 002: ID 05e3:0610 Genesys Logic, Inc. Hub
    Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 002 Device 002: ID 0781:5590 SanDisk Corp. Ultra Dual
    Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 001 Device 005: ID 0781:5566 SanDisk Corp. Cruzer Slice
    Bus 001 Device 002: ID 1a40:0101 Terminus Technology Inc. Hub
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    
    root@rock-5-itx:~# lsusb -t
    /:  Bus 08.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
    |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=uas, 5000M
    /:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
            |__ Port 1: Dev 5, If 0, Class=Mass Storage, Driver=usb-storage, 5000M
            |__ Port 2: Dev 6, If 0, Class=Mass Storage, Driver=uas, 5000M
            |__ Port 3: Dev 7, If 0, Class=Mass Storage, Driver=usb-storage, 5000M
            |__ Port 4: Dev 3, If 0, Class=Mass Storage, Driver=uas, 5000M
    /:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
    /:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
    /:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
    /:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=usb-storage, 480M
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
            |__ Port 4: Dev 5, If 0, Class=Mass Storage, Driver=usb-storage, 480M
    
    root@rock-5-itx:~# sbc-bench.sh -S
      * 111.8GB "Samsung SSD 750 EVO 120GB" SSD as /dev/sda [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind ASMedia SATA 6Gb/s bridge (174c:55aa), 3% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps), drive temp: 26°C
      * 111.8GB "Samsung SSD 840 EVO 120GB" SSD as /dev/sdb [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind JMicron JMS567 SATA 6Gb/s bridge (152d:3562), 3% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps), drive temp: 25°C
      * 115.7GB "SanDisk Corp. Ultra Fit" as /dev/sdc: USB, Driver=usb-storage, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps)
      * 7.3TB "TOSHIBA HDWF180" HDD as /dev/sdd [SATA 3.3, 6.0 Gb/s (current: 6.0 Gb/s)]: behind ASMedia SATA 6Gb/s bridge (174c:55aa), Driver=usb-storage, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps), drive temp: 21°C
      * 115.7GB "SanDisk Corp. Ultra Dual" as /dev/sde: USB, Driver=usb-storage, 480Mbps (capable of 12Mbps, 480Mbps, 5Gbps)
      * 7.5GB "SanDisk Corp. Cruzer Slice" as /dev/sdf: USB, Driver=usb-storage, 480Mbps
      * 111.8GB "Samsung SSD 840 EVO 120GB" SSD as /dev/sdg [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind VIA Labs VL715/VL716 SATA 6Gb/s bridge (2109:0715), 5% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps, 10Gb/s Symmetric RX SuperSpeedPlus, 10Gb/s Symmetric TX SuperSpeedPlus), drive temp: 23°C

</details>

## Display capabilities

Disclaimer: not my area since I usually operate SBC headless as such just a quick list copy&pasted from Radxa:

  * DisplayPort Alt mode available at the USB-C port (4Kp60)
  * HDMI 8Kp60 available native by the SoC
  * HDMI 4K available via Radxa RA620-1 DisplayPort-to-HDMI converter 
  * eDP FPC connector for 4Kp60 LCD panel
  * up to 2 x four-lane MIPI DSI connectors

Six displays in total but according to Radxa only four can be used concurrently.

## Video input capabilities

  * HDMI up to 4Kp60
  * up to 2 x four-lane MIPI CSI connectors (switching between CSI and DSI can be done by a device-tree overlay)

## TF card slot and eMMC

`sbc-bench -S` reports the two SDIO attached MMC storage types as follows:

  * 238.8GB "Samsung EE4S5" UHS SDR104 SDXC card as /dev/mmcblk1: date 05/2023, manfid/oemid: 0x00001b/0x534d, hw/fw rev: 0x3/0x0
  * 29.1GB "Samsung BJTD4R" HS400 Enhanced strobe eMMC card as /dev/mmcblk0: date 09/2023, manfid/oemid: 0x000015/0x0100, hw/fw rev: 0x0/0x0300000000000000

The TF card implementation is SDR104 / UHS-I capable so let's benchmark my 256GB Samsung EVO card in the slot using `iozone -e -I -a -s 100M -r 4k -r 16384k -i 0 -i 1 -i 2`:

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          102400       4     2738     2762    12852    12789    10197     2776
          102400   16384    65483    65060    88313    88555    88551    64214

Results confirm SDR104 / UHS-I with 65/88 MB/s write/read since most probably the TF card is the bottleneck here so the theoretical 104 MB/s can't be reached but the 50MB/s SDR50 mode would provide are clearly exceeded.

The 32GB eMMC on the dev sample is empty but according to Radxa's cardboard box that may change when Rock 5 ITX can be bought since 'On board 32G eMMC for ROOBI OS' is printed on the retail package. As such the board might boot up from the prepopulated eMMC and provide a way for an OS to be installed from the Internet?

For now let's check the perforance of the eMMC module again with `iozone -e -I -a -s 100M -r 4k -r 16384k -i 0 -i 1 -i 2` on an ext4 partition:

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          102400       4    26175    35219    25977    25981    25705    34631
          102400   16384   108049   108512   294709   295442   295918   108310

Great random IOPS at 4K and 100/300 MB/s at sequential transfer speeds. Both nice.

## USB-C port

This connector is not meant for powering but provides only DisplayPort and USB3 OTG or host. The latter is the default with Radxa's OS images as such I connected a Samsung EVO 840 in an USB3 disk enclosure and measured via `iozone -e -I -a -s 500M -r 4k -r 16384k -i 0 -i 1 -i 2`:

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          512000       4    21340    28102    22988    22459    13014    22292
          512000   16384   385996   365681   386597   386635   386580   385638

385 MB/s at 16M blocksize are OK since this benchmark was on a btrfs filesystem (with a bit more overhead due to copy-on-write).

To confirm the USB-C port *not* having to share bandwidth with the four USB3-A receptacles I set up an `/dev/md0` as raid0 with another EVO750, formatted it with ext4 and let again an `iozone -e -I -a -s 500M -r 4k -r 16384k -i 0 -i 1 -i 2` run:

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          512000       4    23749    31631    32728    32618    16693    31494
          512000   16384   747561   747706   735573   739497   728055   745962

Results as expected (sequential bandwidth almost doubled, 4K random IOPS also improved)

<details>
  <summary>lsusb / sbc-bench -S / mdadm --detail</summary>

    root@rock-5-itx:/home/radxa# lsusb
    Bus 006 Device 003: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
    Bus 006 Device 002: ID 05e3:0620 Genesys Logic, Inc. USB3.2 Hub
    Bus 006 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 005 Device 002: ID 05e3:0610 Genesys Logic, Inc. Hub
    Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 008 Device 002: ID 2109:0715 VIA Labs, Inc. VL817 SATA Adaptor
    Bus 008 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 007 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 001 Device 002: ID 1a40:0101 Terminus Technology Inc. Hub
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    
    root@rock-5-itx:/home/radxa# lsusb -t
    /:  Bus 08.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
        |__ Port 1: Dev 2, If 0, Class=Mass Storage, Driver=uas, 5000M
    /:  Bus 07.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
    /:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
            |__ Port 3: Dev 3, If 0, Class=Mass Storage, Driver=uas, 5000M
    /:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
    /:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
    /:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
    /:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
    
    root@rock-5-itx:/home/radxa# sbc-bench.sh -S
      * 111.8GB "Samsung SSD 750 EVO 120GB" SSD as /dev/sda [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind ASMedia SATA 6Gb/s bridge (174c:55aa), 3% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps), drive temp: 32°C
      * 111.8GB "Samsung SSD 840 EVO 120GB" SSD as /dev/sdb [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind VIA Labs VL715/VL716 SATA 6Gb/s bridge (2109:0715), 3% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps, 10Gb/s Symmetric RX SuperSpeedPlus, 10Gb/s Symmetric TX SuperSpeedPlus), drive temp: 31°C
    
    root@rock-5-itx:/home/radxa# mdadm --detail /dev/md0
    /dev/md0:
               Version : 1.2
         Creation Time : Mon Apr 15 18:20:48 2024
            Raid Level : raid0
            Array Size : 234305536 (223.45 GiB 239.93 GB)
          Raid Devices : 2
         Total Devices : 2
           Persistence : Superblock is persistent
    
           Update Time : Mon Apr 15 18:20:48 2024
                 State : clean 
        Active Devices : 2
       Working Devices : 2
        Failed Devices : 0
         Spare Devices : 0
    
                Layout : -unknown-
            Chunk Size : 512K
    
    Consistency Policy : none
    
                  Name : rock-5-itx:0  (local to host rock-5-itx)
                  UUID : 8b6e75db:054fa470:02437d6c:3a269f87
                Events : 0
    
        Number   Major   Minor   RaidDevice State
           0       8       17        0      active sync   /dev/sdb1
           1       8        1        1      active sync   /dev/sda1

</details>

And since we're at it by moving the EVO840 from the USB-C port to one of the other USB3-A receptacles we can quickly confirm with very same `/dev/md0` that all the USB3-A ports behind the onboard USB3 hub have to share bandwidth:

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          512000       4    19329    26530    27105    27171    16212    23857
          512000   16384   405432   400813   374786   375842   374552   402061

<details>
  <summary>lsusb / sbc-bench -S</summary>

    root@rock-5-itx:/home/radxa# lsusb
    Bus 006 Device 004: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
    Bus 006 Device 003: ID 152d:3562 JMicron Technology Corp. / JMicron USA Technology Corp. JMS567 SATA 6Gb/s bridge
    Bus 006 Device 002: ID 05e3:0620 Genesys Logic, Inc. USB3.2 Hub
    Bus 006 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 005 Device 002: ID 05e3:0610 Genesys Logic, Inc. Hub
    Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 003 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
    Bus 001 Device 002: ID 1a40:0101 Terminus Technology Inc. Hub
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    
    root@rock-5-itx:/home/radxa# lsusb -t
    /:  Bus 06.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 5000M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 5000M
            |__ Port 2: Dev 3, If 0, Class=Mass Storage, Driver=uas, 5000M
            |__ Port 3: Dev 4, If 0, Class=Mass Storage, Driver=uas, 5000M
    /:  Bus 05.Port 1: Dev 1, Class=root_hub, Driver=xhci-hcd/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
    /:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
    /:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=ohci-platform/1p, 12M
    /:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
    /:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-platform/1p, 480M
        |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/4p, 480M
        
    root@rock-5-itx:/home/radxa# sbc-bench.sh -S
      * 111.8GB "Samsung SSD 840 EVO 120GB" SSD as /dev/sda [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind JMicron JMS567 SATA 6Gb/s bridge (152d:3562), 5% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps), drive temp: 32°C
      * 111.8GB "Samsung SSD 750 EVO 120GB" SSD as /dev/sdb [SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)]: behind ASMedia SATA 6Gb/s bridge (174c:55aa), 5% worn out, Driver=uas, 5Gbps (capable of 12Mbps, 480Mbps, 5Gbps), drive temp: 33°C
    
</details>

Now trying to use the USB-C port as an el cheapo network device just as [I managed it with RPi 5B recently](https://github.com/raspberrypi/linux/issues/5737#issuecomment-1943440662). First step is executing `rsetup` and choosing the right device-tree overlay 'Set OTG port 0 to Peripheral mode' and then enabling the Ethernet USB gadget:

![Choosing device-tree overlays with rsetup](../media/rock5-itx-rsetup-dtbos.png)

After installing `avahi-autoipd` and creating `/etc/network/interfaces.d/usb0` with appropriate contents an `usb0` device with link local addresses appeared and `ip a` shows:

    4: usb0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state DOWN group default qlen 1000
        link/ether de:3c:fc:30:d3:0c brd ff:ff:ff:ff:ff:ff
        inet 169.254.10.114/16 brd 169.254.255.255 scope link usb0:avahi
           valid_lft forever preferred_lft forever

But nothing on the other end of the USB-C cable to be seen so am going to revisit this soon with a more mature OS image from Radxa.

## PoE via optional PoE module

Both RJ45 jacks are PoE enabled but for 'Power over Ethernet' to be really working Radxa's PoE module must be seated between ATX power connector and RJ45 jacks . When connected to a PoE switch a short press on the power button is needed for the board to boot.

Measuring the 12V rail on the SATA power ports gives 11.64V and when checking the DC voltage generated by the PoE module via SARADC a 11.7V value confirms the voltage being a bit on the low side for HDDs. Radxa chose to [keep Rock 5 ITX compatible to 5B wrt DC-IN voltage measurement](https://forum.radxa.com/t/realtime-power-usage/15027/11?u=tkaiser):

`awk '{printf ("%0.2f",$1/173.5); }' </sys/devices/iio_sysfs_trigger/subsystem/devices/iio\:device0/in_voltage6_raw`.

![PoE module](../media/rock5-itx-5.jpg)

## SATA power ports

For each of the 4 SATA connectors there's also a power connector carrying 12V, GND, GND and 5V ([Floppy connector](https://en.wikipedia.org/wiki/Berg_connector)). Powering the board via DC-IN from my ODROID SmartPower 3 I chose 12.4V and 11.6V to compare with SARADC values and Multimeter readings.

At the 12.4V setting both SmartPower and SARADC report 12.37V and the Multimeter measures 12.29V at the 12V rail of all four power ports.

At the 11.6V setting SmartPower/SARADC report 11.57V while the Multimeter reads 11.56V on all four power ports. That means the 12V power rail is directly connected to DC-IN be it the PoE mpdule, ATX or the DC-IN jack.

The 5V rail in both settings shows 5.16V - 5.17V since behind DC-DC circuitry.

So in case you want to power 5.25" HDDs (or those old 3.5" WD Velociraptor 10.000rpm HDDs that also need juice on both 5V and 12V rail) you need to be careful about your 12V power source and may run into problems with 5.25" HDDs with PoE. But I guess most people combining the board with spinning rust will use an ATX PSU anyway and the PSU's SATA power connectors.

![12V rail testing](../media/rock5-itx-12v-rail-testing.jpg)

## SATA performance

Spoiler alert: Not able to test for maximum performance since one of my old/crappy SATA SSDs just died and from the three remaining two are not able to saturate SATA 6Gpbs anyway. As such this is just preparation for an upcoming SMB multichannel test with 2 x 2.5GbE.

Three 120/128 GB Samsung SSDs are connected to the SATA ports all externally powered:

    root@rock-5-itx:/home/radxa# sbc-bench.sh -S
      * 111.8GB "Samsung SSD 750 EVO 120GB" SSD as /dev/sda: SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s), 3% worn out, drive temp: 26°C
      * 111.8GB "Samsung SSD 840 EVO 120GB" SSD as /dev/sdb: SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s), 3% worn out, drive temp: 26°C
      * 119.2GB "SAMSUNG MZ7TE128HMGR-00004" SSD as /dev/sdc: SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s), 3% worn out, drive temp: 27°C

<details>
  <summary>Creating an mdadm raid0 out of them and formatting the array with ext4</summary>

    root@rock-5-itx:/home/radxa# wipefs --all --force /dev/sda?; sudo wipefs --all --force /dev/sda
    /dev/sda1: 4 bytes were erased at offset 0x00001000 (linux_raid_member): fc 4e 2b a9
    /dev/sda: 8 bytes were erased at offset 0x00010040 (btrfs): 5f 42 48 52 66 53 5f 4d
    /dev/sda: 8 bytes were erased at offset 0x00000200 (gpt): 45 46 49 20 50 41 52 54
    /dev/sda: 8 bytes were erased at offset 0x1bf2975e00 (gpt): 45 46 49 20 50 41 52 54
    /dev/sda: 2 bytes were erased at offset 0x000001fe (PMBR): 55 aa
    
    root@rock-5-itx:/home/radxa# wipefs --all --force /dev/sdb?; sudo wipefs --all --force /dev/sdb
    /dev/sdb1: 4 bytes were erased at offset 0x00001000 (linux_raid_member): fc 4e 2b a9
    /dev/sdb: 8 bytes were erased at offset 0x00010040 (btrfs): 5f 42 48 52 66 53 5f 4d
    /dev/sdb: 8 bytes were erased at offset 0x00000200 (gpt): 45 46 49 20 50 41 52 54
    /dev/sdb: 8 bytes were erased at offset 0x1bf2975e00 (gpt): 45 46 49 20 50 41 52 54
    /dev/sdb: 2 bytes were erased at offset 0x000001fe (PMBR): 55 aa
    
    root@rock-5-itx:/home/radxa# wipefs --all --force /dev/sdc?; sudo wipefs --all --force /dev/sdc
    /dev/sdc1: 2 bytes were erased at offset 0x00000438 (ext4): 53 ef
    /dev/sdc: 8 bytes were erased at offset 0x00010040 (btrfs): 5f 42 48 52 66 53 5f 4d
    /dev/sdc: 8 bytes were erased at offset 0x00000200 (gpt): 45 46 49 20 50 41 52 54
    /dev/sdc: 8 bytes were erased at offset 0x1dcf855e00 (gpt): 45 46 49 20 50 41 52 54
    /dev/sdc: 2 bytes were erased at offset 0x000001fe (PMBR): 55 aa
    
    root@rock-5-itx:/home/radxa# for i in a b c ; do sgdisk -n 1:0:0 /dev/sd${i} ; done
    Creating new GPT entries in memory.
    Warning: The kernel is still using the old partition table.
    The new table will be used at the next reboot or after you
    run partprobe(8) or kpartx(8)
    The operation has completed successfully.
    Creating new GPT entries in memory.
    Warning: The kernel is still using the old partition table.
    The new table will be used at the next reboot or after you
    run partprobe(8) or kpartx(8)
    The operation has completed successfully.
    Creating new GPT entries in memory.
    The operation has completed successfully.
    
    root@rock-5-itx:/home/radxa# mdadm --create --verbose /dev/md0 --level=0 --raid-devices=3 /dev/sd[a-c]1
    mdadm: chunk size defaults to 512K
    mdadm: Defaulting to version 1.2 metadata
    mdadm: array /dev/md0 started.
    
    root@rock-5-itx:/home/radxa# mkfs.ext4 /dev/md0
    mke2fs 1.46.2 (28-Feb-2021)
    /dev/md0 contains a ext4 file system
    	last mounted on /mnt on Mon Apr 15 18:35:12 2024
    Proceed anyway? (y,N) Y
    Discarding device blocks: done                            
    Creating filesystem with 89818112 4k blocks and 22462464 inodes
    Filesystem UUID: 93368439-82ee-4f72-823f-2150ff555756
    Superblock backups stored on blocks: 
    	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
    	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (262144 blocks): done
    Writing superblocks and filesystem accounting information: done     

</details>

This results in these performance numbers again measured with `iozone -e -I -a -s 500M -r 4k -r 16384k -i 0 -i 1 -i 2` (BS scores of course since ext4 works in the background due to delayed allocation and my SSDs partially being crap anyway):

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          512000       4    29635    58841    60085    60425    18696    55617
          512000   16384   393731   395017  1041987  1055413  1037428   394694

Well, regardless of SSD crappiness this sucks since sequential write performance is far away from what it should be. Testing again only 16M blocksize after setting every governor and ASPM to `performance` and adjusting IRQ affinity of `ITS-MSI 143130624 Edge ahci[0001:11:00.0]` to an A76 core write 'performance' still sucks:

                                                              random    random
              kB  reclen    write  rewrite    read    reread    read     write
          512000   16384   403808   404209  1104210  1107276  1102140   403876

We're facing a serious problem here for people wanting to combine several SATA SSDs with Rock 5 ITX.

## Power button

When the board is powered via PoE it does not automatically boot when power is present on one of the RK45 ports but it needs a short press of the power button (to be wired from the front-panel header). In contrast to that when powering through the DC-IN jack the board immediately boots when power is available.

A different situation is the board being shut down before. In this mode (powered off but still connected to power source) it consumes below 0.25W - 0.3W and awaits a press on the power button to boot again.

When the power button is pressed for a longer amount of time (~4 seconds?) the board immediately powers off.

## Open questions

  * DC-IN voltage range? As far as I understood so far the 12V requirement is solely related to SATA power (12V rail only needed with 5.25" and some exotic 3.5" HDDs)
  * LPDDR5 modules should be faster than the LPDDR4X on Rock 5B (4224 vs. 5472 MT/s) but with today's boot BLOBS memory bandwidth with LPDDR5 hasn't improved and latency got worse. Why?

## TODO TK

  * investigate LPDDR5 initialization
  * consumption figures (disabling ASM1164, checking through governors/policies and Gen2 vs. Gen3)
  * educational measurement how wasteful ATX PSUs are
  * storage testing (only crappy NVMe SSD lying around, not enough SATA SSDs here)
  * network testing: SMB Multichannel, iperf3, investigating macOS Finder crappiness
  * to be continued
