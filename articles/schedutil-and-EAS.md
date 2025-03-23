# The `schedutil` governor and energy aware scheduling

A quick look on at how broken things are on Linux (on ARM).

With hybrid designs (big.LITTLE/DynamIQ on ARM, on Intel [some SKUs starting with Lakefield](https://github.com/ThomasKaiser/sbc-bench/blob/fe319a30f0662797169eff66feca54764df74599/sbc-bench.sh#L2234-L2400)) the scheduler needs to know both energy consumption at specific clockspeeds as well as performance characteristics to decide when a task should be running on an efficiency and when on a performance core to balance performance and consumption.

The only Linux scheduler able to deal with this 'energy aware scheduling' (EAS) concept is `schedutil` and as such it should be the only reasonable choice on hybrid CPU/SoC designs. Though what sounds great in theory doesn't seem to work that great in practice.

Linux on ARM uses device-tree to configure this stuff.

## `capacity-dmips-mhz` property

This *should* tell the scheduler how performant a specific core is.

In 2002 [ARM explained in detail why using Dhrystone 'today' (in 2002) is 100% wrong](https://www.docjava.com/courses/cr346/data/papers/DhrystoneMIPS-CriticismbyARM.pdf). Money quote: 'Code profiling demonstrates that Dhrystone is not a particularly good predictor of real-world performance, especially for complex embedded systems.'

The very same whitepaper points multiple times to a better benchmarking alternative called Coremark hosted by the [EEMBC benchmarking consortium](https://www.eembc.org).

14 years later two different ARM employees [introduce the `capacity-dmips-mhz` device-tree property](https://github.com/torvalds/linux/commit/de42fe116dcc157b08d5db367bde4742d4e76af3). They point out a ['suitable benchmark'](https://github.com/torvalds/linux/blob/master/Documentation/devicetree/bindings/cpu/cpu-capacity.txt#L18-L25) is needed just to contradict that a few sentences later by this: ['For the time being we however advise usage of the Dhrystone benchmark'](https://github.com/torvalds/linux/blob/master/Documentation/devicetree/bindings/cpu/cpu-capacity.txt#L33).

Just a quick summary what was known about Dhrystone back in 2016 already (simply quoting Wikipedia and ARM's own 'performance benchmarking whitepaper' from 14 years earlier):

  * Dhrystone features unusual code that is not usually representative of modern real-life programs.
  * Dhrystone is susceptible to compiler optimizations. For example, it does a lot of string copying in an attempt to measure string copying performance. However, the strings in Dhrystone are of known constant length and their starts are aligned on natural boundaries, two characteristics usually absent from real programs. Therefore, an optimizer can replace a string copy with a sequence of word moves without any loops, which will be much faster. This optimization consequently overstates system performance, sometimes by more than 30%.
  * Dhrystone's small code size may fit in the instruction cache of a modern CPU, so that instruction fetch performance is not rigorously tested. Similarly, Dhrystone may also fit completely in the data cache, thus not exercising data cache miss performance. To counter fits-in-the-cache problem, the SPECint benchmark was created *in 1988* to include a suite of (initially 8) much larger programs (including a compiler) which could not fit into L1 or L2 caches *of that era*.
  * Dhrystone numbers actually reflect the performance of the C compiler and libraries, probably more so than the performance of the processor itself
  * Dhrystone’s execution is largely spent in standard C library functions, such as strcmp(),strcpy(), and memcpy(). Compiler vendors generally provide these libraries that are typically optimized and hand-written in assembly language. While you may think you are benchmarking a processor, what you are really benchmarking are the compiler writer's optimizations of the C library functions for a particular platform

So why chose ARM in 2016 `capacity-dmips-mhz` over `capacity-coremark-mhz` and none of the kernel guys opposed to that advise? Maybe since nobody gives a sh*t about such stuff?

What about ARM customers? Do they follow ARM's recommendation from 2016 to rely on this Dhrystone joke from last century? [Unfortunately yes](https://lore.kernel.org/linux-devicetree/5A3CD286.2010705@hisilicon.com/T/). I guess at SoC vendors also nobody gives a sh*t about such stuff?

## `dynamic-power-coefficient` property

This DT property for each CPU core *should* tell how much energy this CPU core needs which is an interesting attempt given that modern CPUs usually implement 'dynamic voltage and frequency scaling' (DVFS) and consumption increases a lot more at the highest DVFS OPP so a single number for the whole CPU sounds a little bit strange. But the formula used in the energy model `power (uW) = dynamic-power-coefficient * uV^2 * Freq (Mhz)` may compensate for that (uV^2). Has anyone ever tested this?

[ARM's calculation approach as an example](https://patchwork.ozlabs.org/project/linux-imx/patch/20190128165522.31749-7-quentin.perret@arm.com/). Relies this time on `sysbench` and taking the arithmetic mean of the various values.

`sysbench` is 'famous' for running 15 times faster on same ARMv8 hardware when built for ARMv8 instead of ARMv7 but most probably nobody at ARM knows this. And why SoC vendors should measure performance with Dhrystone but consumption with sysbench is another area of confusion or simply again a sign of 'nobody gives a sh*t about such stuff'?

Does really nobody care? Nope, a developer called Danny Lin chose a better approach [when submitting these 'power and performance' properties for Qualcomm SM8150](https://github.com/torvalds/linux/commit/5b2dae72187de25a90f245482281a9ed0ffd268f). He wrote even an own tool called [freqbench](https://github.com/kdrag0n/freqbench) to automatically collect this data relying on [Coremark](https://www.eembc.org/coremark/).

When looking at this commit ([or **any** other measurements for Qualcomm SoCs](https://github.com/kdrag0n/freqbench/tree/master/results)) one thing always can be observed: the power efficiency for ARM's big cores gets worse with higher clockspeeds (DVFS at work) so I wonder whether averaging all values will not simply result in the big cores getting a `dynamic-power-coefficient` number too high (meaning efficiency being too low)?

How do others do this? Here an [example from Rockchip for the two A72 in RK3399](https://patchwork.kernel.org/project/linux-arm-kernel/patch/1500974575-2244-1-git-send-email-wxt@rock-chips.com/). According to the commit description they measured six OPP between 24 MHz and 408 MHz and then calculated the 'arithmetic mean'. For a core that is supposed to run between 408 MHz and 1800 MHz where we could expect the power efficiency to be much worse. The A53 remained at `100` for which reason exactly? 

At least their BSP kernel contains exact measurements ([rk3399-sched-energy.dtsi](https://github.com/armbian/linux-rockchip/blob/rk-5.10-rkr6/arch/arm64/boot/dts/rockchip/rk3399-sched-energy.dtsi)) but somehow this file neither made its way upstream into mainline within the last 8 years nor is it referenced in any kernel version.

What about other SoC makers? Since I'm currently [evaluating an ARM thingy with A55 and A76 cores](Quick_Preview_of_ROCK_5_ITX.md) let's focus on this combination and look how performant these two core types are according to the respective SoC vendors (who will have done some serious testing, right?):

### [RK3588](https://github.com/armbian/linux-rockchip/blob/rk-5.10-rkr6/arch/arm64/boot/dts/rockchip/rk3588s.dtsi) (A76 1.9 times faster than A55):

  * A55: `capacity-dmips-mhz = <530>;` `dynamic-power-coefficient = <100>;`
  * A76: `capacity-dmips-mhz = <1024>;` `dynamic-power-coefficient = <300>;`

### [Amlogic S928X](https://github.com/CoreELEC/linux-amlogic/blob/amlogic-5.4.210/arch/arm64/boot/dts/amlogic/mesons5.dtsi) (A76 *exactly* two times faster)

  * A55: `capacity-dmips-mhz = <512>;` + `dynamic-power-coefficient = <1024>;`
  * A76: `capacity-dmips-mhz = <1024>;` + `dynamic-power-coefficient = <512>;`

### [Unisoc UMS9620](https://github.com/realme-kernel-opensource/realme_C31_C35_narzo50A-Prime-AndroidR-kernel-source/blob/79c7c8b238a20393a78ee5f1110991bd4280e143/arch/arm64/boot/dts/sprd/ums9620.dtsi) (A76 2.45 times faster than A55)

  * A55: `capacity-dmips-mhz = <417>;` `sched-energy-costs = <&CPU_COST_0 &CLUSTER_COST_0>;`
  * A76: `capacity-dmips-mhz = <1022>;` `sched-energy-costs = <&CPU_COST_1 &CLUSTER_COST_1>;`

### [Google G1](https://github.com/torvalds/linux/blob/master/arch/arm64/boot/dts/exynos/google/gs101.dtsi) (A76 2.5 times faster than A55):

  * A55: `capacity-dmips-mhz = <250>;` `dynamic-power-coefficient = <70>;`
  * A76: `capacity-dmips-mhz = <620>;` `dynamic-power-coefficient = <284>;`
  * X1: `capacity-dmips-mhz = <1024>;` `dynamic-power-coefficient = <650>;`

### [MediaTek MT8186](https://github.com/torvalds/linux/blob/master/arch/arm64/boot/dts/mediatek/mt8186.dtsi) (A76 2.68 times faster than A55):

  * A55: `capacity-dmips-mhz = <382>;` `dynamic-power-coefficient = <84>;`
  * A76: `capacity-dmips-mhz = <1024>;` `dynamic-power-coefficient = <335>;`

Funny! A76 is between 1.9 to 2.7 times faster than A55 at identical clockspeed.

The most fascinating properties are those from Amlogic of course: they simply decided to use a nicely looking `1024` number, divide it by 2 for fun and then throw those two numbers at random DT locations which results in the A76 in their S928X being exactly twice as performant **and** power efficient at the same time as a little A55. 'As expected' one could say since Amlogic has a 'track record' of not only [cheating with clockspeeds](https://www.cnx-software.com/2016/08/28/amlogic-s905-and-s912-processors-appear-to-be-limited-to-1-5-ghz-not-2-ghz-as-advertised/) but also messing up performance on SoCs with more than one cluster since in the past they always forgot to add `capacity-dmips-mhz` properties for the cores and as an example [on their S912 with two clusters consisting of A53 limited to either 1.0 or 1.4 GHz single-threaded workloads mostly ended up on the slower A53 at only 1.0 GHz](https://forum.khadas.com/t/s912-limited-to-1200-mhz-with-multithreaded-loads/2311/54?u=tkaiser).

Google/Samsung and MediaTek seem to have actually measured something while the 100/300 values from Rockchip for the RK3588 are obviously just two random numbers chosen. 

The winner is Unisoc since they [provide detailed power costs](https://github.com/realme-kernel-opensource/realme_C31_C35_narzo50A-Prime-AndroidR-kernel-source/blob/79c7c8b238a20393a78ee5f1110991bd4280e143/arch/arm64/boot/dts/sprd/ums9620.dtsi#L133-L173) though nobody knows with which methodology they generated those numbers (`sysbench`?)

But why do the `capacity-dmips-mhz` differ between SoC vendors? Did they by accident discover that using Dhrystone is a bad idea since being more of a software than a hardware benchmark not representing anything even remotely what we do on today's CPUs?

At least not in Rockchip's case since I personally measured half a year ago to discover this: [the VAX MIPS ratings generated with same dhrystone binary suggest the A76 being 2.32 faster than an A55 at same clockspeed (14540 / 6260 = 2.32)](https://github.com/ThomasKaiser/sbc-bench/blob/master/Benchmarking_some_benchmarks.md#dhrystone--dmips--dmipsmhz). So when relying on this DMIPS nonsense it would have to read `capacity-dmips-mhz = <441>;` for the A55 cores. And this should be the same for any other A76/A55 combo out there since this Dhrystone joke is that small that it runs entirely inside CPU caches. But since the Dhrystone joke is a compiler/libs benchmark it may be possible that Rockchip really set these numbers based on 'measured' DMIPS just by using a different compiler version or whatever.

Let's use the popular Geekbench 6 suite (though giving questionable numbers on any platform other than x86) and have a look at the A76 and A55 both at 1800 MHz in RK3588 (with the LPDDR at 4800 MT/s): `MaxKHz=1800000 Netio=powerbox-1/4 sbc-bench.sh -G` (sbc-bench executes Geekbench multiple times and on hybrid systems also on each CPU cluster individually)

A76 single-threaded *on average* 3.23 times faster than A55 at same clock: https://browser.geekbench.com/v6/cpu/compare/5841948?baseline=5842066 ([full results](https://sprunge.us/B7m0Gs))

Clicking on the Geekbench browser link is interesting due to the wide range of performance differences with individual sub tests between the in-order A55 and the out-of-order A76 which hints at the whole idea of an energy model based on two properties per core type being flawed (I guess there's lots of real-world tasks where execution on an A55 needs way more time than just 2.5 times longer according to what SoC vendors have written to `.dtsi` files)

But even if the whole concept would work flawlessly ARM's Dhrystone recommendation ruins it already (since suggesting little cores being a lot faster than they actually are) and SoC vendors writing funny numbers in DT nodes doesn't help either.

Seems nobody gives a sh*t about such stuff?

Later addition: I've been in touch with Rockchip engineers and for their 1.9 performance ratio they wrote:

这个和dhrystone/coremark的版本，编译器选项的关系很大，我之前用的coremark测试出来结果是1.9倍左右。实际上，还要考虑真实的使用情况，A76/A55性能比较无法给出一个很好的量化值。

(This has a lot to do with the version of dhrystone/coremark, compiler options, the coremark test I used before came out around 1.9x. In reality, there are real world usage situations to consider, and the A76/A55 performance comparison doesn't give a good quantitative value.)



