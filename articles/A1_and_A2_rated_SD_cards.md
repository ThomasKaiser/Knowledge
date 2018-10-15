# Performance impact of MMC storage used on SBC

## Historical background

SD cards were initially made for pure sequential workloads (storing/reading pictures or video). The vast majority of cards was only optimized for this single use case and performed extremely poorly with random IO access patterns.

When using SD cards as main storage on SBC (single board computers) this limitation resulted in poor performance since the main storage access pattern when running Linux or Android is random IO (accessing mostly small chunks of data in a random fashion in contrary to sequentially writing/reading large chunks of data in cameras and video recorders).

SD Association's 'speed classes' only dealt with sequential IO (minimum MB/s ratings) and it was an adventure to find SD cards that also perform ok-ish or even great with random IO access patterns as it's normal with Android/Linux. See here for a historical overview trying to find good SD cards for the use case 2 1/2 years ago: [https://forum.armbian.com/topic/954-sd-card-performance/](https://forum.armbian.com/topic/954-sd-card-performance/)

The stuff we do mostly on SBC requires high *random* IO but is not that much dependent on *sequential* performance unless you stream data (the NAS use case for example). So running off storage with superior random IO performance greatly improves the time needed to boot, browse the web (browser do constant random IO in the background), install updates and so on.

## 'Application Performance Class' to the rescue

Some time ago SD Association tried to address this problem and defined also specs and 'speed classes' for random IO access patterns mainly with Android in mind: the [Application Performance Classes](https://www.sdcard.org/developers/overview/application/index.html) A1 and A2 were invented.

![](https://www.sdcard.org/developers/overview/application/img/img_application.jpg)

### Application Performance Class 1 (A1)

This performance class requires at least 1500/500 read/write IOPS (IO operations per second) with a 4k blocksize (small data chunks) and at least 10 MB/s sustained sequential write performance. No special host requirements are needed, the card simply has to exceed the performance requirements on its own.

At the end of 2017 we were already able to buy some great performing A1 rated cards that are multiple times faster than what the specs require. E.g. a SanDisk Extreme A1 being two to six times faster than required by A1 specs.

### Application Performance Class 2 (A2)

A2 promises even better performance with 4000/2000 read/write IOPS minimum but there's a problem since as outlined by the SD Association A2 cards show "much higher performance than A1 performance by using functions of Command Queuing and Cache".

Cache and Command Queuing require host (driver) support since the host needs to activate those new features first. The cache feature on A2 rated cards makes use of volatile RAM on the card requiring the host to learn new commands to issue flushing the cache (involving the risk of data losses -- for details see especially chapter 4.17 in [Physical Layer Simplified Specification 6.0](https://www.sdcard.org/downloads/pls/pdf/index.php?p=Part1_Physical_Layer_Simplified_Specification_Ver6.00.jpg&f=Part1_Physical_Layer_Simplified_Specification_Ver6.00.pdf&e=EN_SS1))

### Real world 2018 A1 and A2 performance comparison

I recently acquired two A2 rated SanDisk cards and to my surprise they were both slower than my older SanDisk A1 rated cards. They barely meet A1 specs (the 1500/500 read/write IOPS) and since silly me didn't read the specs before I thought the SanDisk Extreme Plus A2 I bought first would be defective so I returned it to Amazon and bought a SanDisk Extreme Pro A2 instead.

The Extreme Pro is even slightly slower than the Extreme Plus and only now I started to understand that A2 cards can only show their performance if the host is also 'A2 compliant' needing updated MMC drivers to deal with these new cards. It seems at the time of this writing this is still missing in Linux (tested with 4.19.0-rc4).

## Benchmarking the cards

I rely on the following methodology for testing (using [sd-card-bench.sh](https://github.com/ThomasKaiser/sbc-bench/blob/master/sd-card-bench.sh)):

* `iozone` benchmark to test through sequential and random IO at various block sizes from 1KB to 16MB
* Measuring the time until `/etc/rc.local` gets executed as a means of 'boot time performance'
* Installation of a DE (desktop environment) as representation of large software installations or upgrades (~400 packages, dealing with a lot of small files and constant `sync` calls to ensure installation on disk is intact). The task is a mixture of write and read access (10 times more write than read though)
* Removing 150 packages of the DE (again a lot of synced random write IO -- deleting something is write access at the block device layer since only filesystem metadata and structures get updated)
* Reinstalling these 150 packages again, this time the packages come from the local apt cache (still in the page cache so not even involving storage access at all since still in DRAM) so speed of Internet access and upstream servers does not interfere with storage performance

All tests happen on a NanoPi M4 with UHS/SDR104 mode enabled (for reliability reasons limiting SD host controller clock source PLL configuration in a similar way to what [Hardkernel did on ODROID-N1](https://forum.odroid.com/viewtopic.php?f=153&t=30247#p216250) therefore bottlenecking sequential performance to ~66MB/s). All tests done with a freshly built Armbian Stretch using defaults (ext4, 600 sec commit interval).

![](../media/IMG_8122.JPG)

8 different storage types were tested. In brackets the card production date according to the card's metadata:

* Intenso 'Class 4' 4GB (02/2015)
* SanDisk 'Ultra' 8GB (06/2016)
* SanDisk Industrial 8GB (02/2018)
* SanDisk Extreme Plus (07/2015)
* SanDisk Ultra A1 32GB (09/2017)
* SanDisk Extreme A1 32GB (10/2017)
* SanDisk Extreme Pro A2 64GB (08/2018)
* FriendlyELEC's Samsung eMMC 8GB module (02/2018)

The Intenso Class 4 card is used as an equivalent for 'average SD card' SBC users pull out of an old digital camera and use now for their Linux installation.

## Results overview

The links in the title row contain [sd-card-bench](https://github.com/ThomasKaiser/sbc-bench/blob/master/sd-card-bench.sh)'s raw output to check for details. Unfortunately I forgot to upload the logfile for the 'Average' Intenso card and have overwritten the card already in the meantime for a new set of tests:

|            | Average card | [SanDisk Ultra](http://ix.io/1pbI) | [SanDisk Industrial](http://ix.io/1pa0) | [Extreme Plus](http://ix.io/1pa8) | [Ultra A1](http://ix.io/1p8P) | [Extreme A1](http://ix.io/1p98) | [Extreme Pro A2](http://ix.io/1p8K) | [eMMC](http://ix.io/1pc9) |
| ---------: | :-----: | :-----: | :-----: | :-----: | :-----: | :-----: | :-----: | :-----: |
|   1k read IOPS | 1574 | 2243 | 2257 | 2903 | 3262 | 3840 | 2431 | 5191 |
|  1k write IOPS |   99 |  184 |  416 |  564 |  470 |  826 |  417 | 1230 |
|   4k read IOPS | 1926 | 2139 | 1853 | 2344 | 2867 | 3201 | 1766 | 5083 |
|  4k write IOPS |   33 |  133 |  762 |  770 |  787 | 1480 |  732 | 1943 |
|  16k read IOPS |  726 | 1421 | 1571 | 1915 | 1784 | 2030 | 1660 | 3707 |
| 16k write IOPS |    2 |    6 |  457 |  577 |  516 |  769 |  427 |  981 |
|      read MB/s |   43 |   45 |   66 |   66 |   66 |   66 |   65 |  128 |
|     write MB/s |   10 |    8 |   29 |   62 |   18 |   59 |   50 |   32 |
|  boot time (in sec) | 26.7 | 20.9 | 20.7 | 13.4 | 14.2 | 16.2 | 17.0 | 15.5 |
| 1st install (in sec) | 1070 |  430 |  253 |  216 | 207 |  161 |  190 | 178 |
| DE removal (in sec) |  149 |   68 |   49 |   33 |   31 |   28 |   36 |  27 |
| 2nd install (in sec) |  470 |  157 |  113 |   76 |   75 |   57 |   75 |  65 |

## Obvious results

* At the time of this writing due to lacking A2 host support (drivers) at least SanDisk A2 rated cards are outperformed by A1 cards from the same company (even the cheap Ultra A1 is faster everywhere except sequential write performance)
* My two A1 rated cards are from late 2017 so maybe SanDisk adjusted A1 performance in the meantime. A test with recently acquired A1 cards would be needed
* 'Average' SD cards show horribly low random write performance (for whatever reasons especially at 16k block size -- here the slowest card 'performs' over 500 times worse compared to the eMMC module)
* DE (desktop environment) installation time correlates **only** with **random** write and not sequential write performance (compare 'Class 4' with SanDisk Ultra, compare Extreme Plus with Extreme A1 and especially Ultra A1)

## Looking at storage access patterns

`sd-card-bench` runs an `iostat 20` task in the background that monitors storage activity (1st row representing what happened since last boot and then querying the kernel counters every *n* seconds -- in our case *20*). This provides some insights as to what happens in reality. Is the OS busy reading or writing, is it a mix of both, are bottlenecks becoming obvious?

The 2nd column shows the 'tps' (transactions per second, comparable with IOPS), then average throughput numbers are shown in KB/s and then absolute amount of data since last query.

### Booting

The below is the `iostat 20` output with system on the eMMC. First entry is ~16 seconds after the kernel loaded, the subsequent entries are with a 20 sec delay in between:

    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    mmcblk1         214.00     10296.19        69.67     132409        896
    mmcblk1           2.00        67.20         0.00       1344          0
    mmcblk1           0.45         0.20         2.20          4         44
    mmcblk1           0.00         0.00         0.00          0          0

While booting with this Armbian Stretch CLI image ~130 MB are read from the storage and less than 1 MB is written. So this is mostly read storage access and based on the above numbers (comparing the synthetic benchmark results with actual boot times) there are no obvious correlations to random or sequential read performance (why boots SanDisk Extreme Plus or Ultra A1 faster than the eMMC module for example?)

### Installing a Desktop Environment

This is again the eMMC while installing the DE the first time. Packages are downloaded from upstream apt servers (so 'the Internet' is a potential bottleneck here), are then stored on 'disk' and obviously partially read again (write/read ratio in the example below is exactly 10:1)

    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    mmcblk1          95.15      3558.20      4072.60      71164      81452
    mmcblk1         187.70        67.40      7887.80       1348     157756
    mmcblk1         353.85        48.40      5597.20        968     111944
    mmcblk1         300.95        13.60      5735.00        272     114700
    mmcblk1         321.20         2.40      5743.80         48     114876
    mmcblk1         170.75       101.20      5659.00       2024     113180
    mmcblk1         202.30       104.20      6125.40       2084     122508
    mmcblk1         272.20       653.40      3813.60      13068      76272
    mmcblk1         181.05       266.00      3427.80       5320      68556

This task took 178 seconds on the eMMC but a whopping 1070 seconds when running on the 'Class 4' card:

    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    mmcblk0          49.30       599.60       565.00      11992      11300
    mmcblk0           7.35         0.00      3258.40          0      65168
    mmcblk0           8.95         0.00      3768.60          0      75372
    mmcblk0          16.95         3.00      2687.40         60      53748
    mmcblk0          43.45        70.00       776.40       1400      15528
    mmcblk0          26.05         0.00       498.80          0       9976
    mmcblk0          18.25         3.80      1524.20         76      30484
    mmcblk0          26.30         0.80       717.60         16      14352
    mmcblk0          31.80         7.20      1381.80        144      27636
    mmcblk0          21.70         0.40      3002.00          8      60040
    mmcblk0          26.15         0.40       882.20          8      17644
    mmcblk0          30.25         8.20       892.40        164      17848
    mmcblk0          15.55         0.80      1234.80         16      24696
    mmcblk0          34.45         0.40       714.40          8      14288
    mmcblk0          34.55         0.20       900.20          4      18004
    mmcblk0          41.25         0.00       762.20          0      15244
    mmcblk0          20.65         0.80       775.80         16      15516
    mmcblk0          19.70         0.40      1565.60          8      31312
    mmcblk0          28.35         0.00       676.20          0      13524
    mmcblk0          29.40         0.00       903.20          0      18064
    mmcblk0          22.80         8.40       770.80        168      15416
    mmcblk0          26.75         0.80       916.00         16      18320
    mmcblk0          28.80         0.80      1040.80         16      20816
    mmcblk0          20.05         0.00      1108.40          0      22168
    mmcblk0          25.70         0.00       749.60          0      14992
    mmcblk0          21.50         0.00       702.40          0      14048
    mmcblk0          28.60         3.20       905.00         64      18100
    mmcblk0          28.80         0.40      1016.00          8      20320
    mmcblk0          21.45         0.00      1423.60          0      28472
    mmcblk0          21.95         0.20       848.00          4      16960
    mmcblk0          19.95         0.00      1007.20          0      20144
    mmcblk0          24.70         0.80       978.40         16      19568
    mmcblk0          33.40         0.00       991.20          0      19824
    mmcblk0          27.60         0.00       523.00          0      10460
    mmcblk0          28.20         0.00      1291.60          0      25832
    mmcblk0          15.30         0.60      1033.80         12      20676
    mmcblk0          27.90         0.40       743.80          8      14876
    mmcblk0          25.45         0.80      1558.80         16      31176
    mmcblk0          19.40         0.00      2490.60          0      49812
    mmcblk0          33.10         0.20       548.00          4      10960
    mmcblk0          19.90         0.80      4166.80         16      83336
    mmcblk0          17.15         1.20      4157.00         24      83140
    mmcblk0          37.35       538.80      1176.80      10776      23536
    mmcblk0          58.15       250.80      1257.80       5016      25156
    mmcblk0          62.00       297.00      1244.40       5940      24888
    mmcblk0          56.70       155.60       223.60       3112       4472
    mmcblk0          96.45       144.00       994.60       2880      19892
    mmcblk0          96.40        21.80      1354.80        436      27096
    mmcblk0          46.00       186.80       723.20       3736      14464
    mmcblk0          45.95        30.20       692.40        604      13848
    mmcblk0          76.30         0.00       991.00          0      19820
    mmcblk0          51.20         7.60       702.40        152      14048

It's pretty obvious that this 'Class 4' card is limited by its **poor random write performance**: when only writes happen the tps (transactions per second) reported are around ~26 which is (not so) surprisingly slightly below the 4k random write IOPS number the iozone benchmark reported (33 4k read IOPS). Please keep in mind that all these average SD cards get a lot slower with slightly larger data chunks. The 16k random write IOPS with this card are as low as 2 (!) which is just 200 times worse compared to any recent quality card and even 500 times slower than the eMMC module used.

### Removing and reinstalling the Desktop Environment

This task did an `apt purge armbian-stretch-desktop` then measuring the time of the following `apt autoremove` deleting 150 of the installed 400 packages. This is updating apt databases and filesystem structures (file deletion means write activity below the filesystem layer!). Afterwards the time for a 2nd `apt install armbian-stretch-desktop` has been measured which will install the 150 packages again that have been deleted before.

Since this time the packages are already in the local apt cache and also still in the page cache (filesystem cache) neither stuff has to be downloaded from the Inernet nor even read from disk since all the packages are still hold in RAM. That's why we only see write activity this time. First on the eMMC (deletion took 27 sec and 2nd install 65):

    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    mmcblk1         120.55        16.20      3582.20        324      71644
    mmcblk1         149.40         2.00      4477.80         40      89556
    mmcblk1         330.30         0.00      6254.00          0     125080
    mmcblk1         228.45         0.00      6589.20          0     131784
    mmcblk1         141.20         0.00      4244.60          0      84892
    mmcblk1         124.40        26.00      2342.40        520      46848

And the same on the 'Class 4' card again (deletion took 149 sec and 2nd install 470):

    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    mmcblk0          35.50         0.60       551.60         12      11032
    mmcblk0          12.85         0.00      2183.40          0      43668
    mmcblk0          62.25         7.20      1344.20        144      26884
    mmcblk0          45.80         0.00      2583.60          0      51672
    mmcblk0          82.95        10.80      1506.80        216      30136
    mmcblk0          69.10         0.00      1045.40          0      20908
    mmcblk0          66.05         0.00      1504.80          0      30096
    mmcblk0          48.15         0.00      1127.60          0      22552
    mmcblk0          35.70         0.00      1111.00          0      22220
    mmcblk0          15.75         0.00      2151.60          0      43032
    mmcblk0          29.75         0.00       893.20          0      17864
    mmcblk0          29.00         0.00       736.20          0      14724
    mmcblk0          26.00         0.00       887.40          0      17748
    mmcblk0          21.60         0.00       841.40          0      16828
    mmcblk0          19.95         0.00       700.20          0      14004
    mmcblk0          17.60         0.00       797.00          0      15940
    mmcblk0          27.15         0.00       708.00          0      14160
    mmcblk0          24.35         0.00      1018.60          0      20372
    mmcblk0          30.30         0.00       878.40          0      17568
    mmcblk0          18.40         0.00      1397.00          0      27940
    mmcblk0          19.30         0.00       615.60          0      12312
    mmcblk0          21.70         0.00      1033.60          0      20672
    mmcblk0          25.80         0.00       652.00          0      13040
    mmcblk0          27.70         0.00       939.80          0      18796
    mmcblk0          21.30         0.00       634.80          0      12696
    mmcblk0          24.39         0.00       952.32          0      19056
    mmcblk0          13.35         0.00      3945.80          0      78916
    mmcblk0          18.60         0.20      4261.40          4      85228
    mmcblk0          29.40         0.00      1047.00          0      20940
    mmcblk0          57.15         0.00      1326.20          0      26524
    mmcblk0          37.75         0.00       540.00          0      10800
    mmcblk0          69.00         0.00      1105.60          0      22112
    mmcblk0          16.05         0.00      1863.40          0      37268

Again the whole procedure is bottlenecked only by the **poor random write performance** (check the tps numbers vs. KB/s)

## Conclusions

* Buying cheap A1 rated cards makes a lot of sense with typical SBC use cases since random IO performance (especially write) makes the difference between working with these things flawlessly and everything being slow as hell.
* At the time of this writing with missing driver support surprisingly A2 rated cards perform lower than less expensive A1 rated cards. Until A2 host support is ready most probably this will remain the same.

## TODO

* Buy recent A1 rated SanDisk cards and check performance again. Maybe more recent SanDisk A1 cards perform now also lower than the cards measured above that were produced in late 2017 (we've seen this with other vendors already before: e.g. Samsung EVO/EVO+ and even Samsung Pro)
