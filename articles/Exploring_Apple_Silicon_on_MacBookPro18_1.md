# Exploring Apple Silicon on a MacBookPro18,1

** WORK IN PROGRESS **

## Basics

Following last year's [glance on a MacBook Air with M1 SoC](https://github.com/ThomasKaiser/Knowledge/blob/master/articles/Exploring_Apple_Silicon_on_MacBookAir10.md) let's have a look at how things have evolved. In case you're interested how migration from Intel to Apple Silicon works or what is different in general better skim through that text (first).

This year it's about an 16" MBP Pro with 10 CPU and 16 GPU cores (M1 Pro), 16GB LPDDR5 RAM and a 512GB SSD. This thing came with one MagSafe 3 port, a 140W USB-C charger and the appropriate 1.8m cable to interconnect both. There are also three USB-C Thunderbolt 4 ports (still 'just' capable of 40 Gbit/sec but now also with USB4 and allowing to daisychain two 4K displays), one HDMI 2.0 connector, an UHS-II capable SDXC slot (limited to 250MB/sec) and a 3.5mm headphone jack.

Wireless capabilities are provided by a PCIe attached [BCM4387](https://www.techinsights.com/products/bfr-2012-802) (seen first in iPhone 12 Pro Max): BT 5.0 and Wi-Fi 6 (802.11 a/b/g/n/ac/ax) in 2x2 MIMO config.

## Gathering information

### Same procedure as last year

When testing the MacBook Air last year I started 'migrating' everything from last TimeMachine backup of my Intel MBP. Since the Air is gone (used by a colleague for 10 months now) I repeated the process again and it went as flawless as last time: almost everything there and usable, apps open with those windows open on the Intel thing and so on.

To use Rosetta2 in Terminal it's still necessary to start any `x86_64` only GUI app first since only then Rosetta is 'installed' (or maybe it's just usage tracking by Apple?). Also `xcode-select --install` was again needed to get XCode CLI tools running. And this time I trashed my whole homebrew installation migrated from the Intel MacBook directly after a quick 7-zip benchmark (~42500 7-zip MIPS in 'emulation' vs. 52000 native) to start over with a true `arm64` homebrew install.

### Basic collection of information

  * [ioreg -l](https://gist.github.com/ThomasKaiser/24a88bec8e6749bbc723cd6f6e78a3ab)
  * [hidutil list](http://ix.io/3CZh)
  * [system_profiler](http://ix.io/3CZg)

** To be continued, currently waiting for a NETIO PowerBOX 4KF to arrive to do consumption measurements **



