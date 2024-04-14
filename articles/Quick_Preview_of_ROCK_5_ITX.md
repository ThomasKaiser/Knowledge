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

  * 5.5/2.1 mm centre positive DC/IN: maybe wide input range but 12V recommended (directly connected to SATA power ports? TBC)
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
  <summary>lsusb with all six USB receptacles occupied by storage devices</summary>

    root@rock-5-itx:~# lsusb
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

## (stub from here on)

input voltage can be read via SARADC: `awk '{printf ("%0.2f",$1/173.5); }' </sys/devices/iio_sysfs_trigger/subsystem/devices/iio\:device0/in_voltage6_raw`

Board powers on w/o 'power button' pressed

0.3W after `shutdown -h now`. TODO: power button test

LPDDR5 modules should be faster than the LPDDR4X on Rock 5B (4224 vs. 5472 MT/s) but with today's boot BLOBS memory bandwidth with LPDDR5 hasn't improved and latency got worse.

## TODO TK

  * investigate LPDDR5 initialization
  * PoE test wrt DC-IN
  * Measure voltage on SATA power ports
  * USB-C port capabilities
  * consumption figures (disabling ASM1164, checking through governors/policies and Gen2 vs. Gen3)
  * educational measurement how wasteful ATX PSUs are
  * storage testing (only crappy NVMe SSD lying around, not enough SATA SSDs here)
  * network testing: SMB Multichannel, iperf3, investigating macOS Finder crappyness
  * to be continued
