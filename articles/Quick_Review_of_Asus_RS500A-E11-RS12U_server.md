# Quick review of Asus RS500A-E11-RS12U server

This is about an AMD EPIC 7003 1U single-socket server supporting up to 16 DIMM, 12 U.2 NVMe drives, 3 PCIe Gen4 slots, 1 x OCP 3.0 NIC, 2 x M.2 SSDs (SATA or PCIe Gen4) and an AST2600 BMC ([link to pictures, data sheet and parts list](https://servers.asus.com/products/Servers/Rack-Servers/RS500A-E11-RS12U)).

Since we were rather impressed by the energy efficiency of an [Asus PN41 Mini PC](https://www.cnx-software.com/2021/10/15/2-5gbe-mini-pc-asus-pn41/) now serving as a Check_MK server and central rsyslog host operating at ~6.5W and we needed new hypervisors why not give Asus servers a try?

We chose the specific model since taking only 1 rack unit, providing 12 hot-swappable U.2 2.5" bays (+ 4 optional internal 2.5" bays), 2 x M.2 slots for SATA or NVMe boot SSDs, 3 PCIe Gen4 slots, 2 x onboard GbE + 2 x SFP+ 10GbE NICs in OCP slot. Also it's single-socket so in case we ever need to run some commercial software that gets licensed per socket it gets cheaper.

![](../media/rs500a_front.jpg)

## Mainboard / chassis

The server is based on an [Asus KRPA-U16 server/workstation mainboard](https://servers.asus.com/products/Servers/Server-Motherboards/KRPA-U16) equipped with an ASMB10-iKVM (based on an [ASpeed AST2600 BMC](https://www.aspeedtech.com/server_ast2600/)).

Three PCIe Gen4 slots are exposed via riser cards to the back of the machine. If the plastic cover to route airflow from fans 2-4 to the CPU is removed it looks like this:

![](../media/rs500a_without_cover.jpg)



## CPU

We ordered the servers with an EPYC 7232P 8-Core Processor (16 threads) running at 3200 MHz (in Linux cpufreq driver talks about 3100 MHz but single-threaded measured it's ~3190 MHz, see [sbc-bench results](http://ix.io/47Nv)).

Single-threaded performance is on par or slightly better (10%-15%) than our previous workhorse (Xeon Silver 4110 bursting at 3000 MHz) but multi-threaded it's a noticable +50% (55% with 7-zip, %70 Passmark score). Given both CPUs have same count of cores/threads this hints at the Intel chip not holding burst frequencies when all cores are fully loaded due to power constraints. 

Looking at [sbc-bench results for Xeon 4110](http://ix.io/484u) confirms this: cores end up sometimes at 2100 MHz (the base clock to which the 85W TDP rating applies) while the AMD Zen2 cores remain at 3100 MHz with sustained high loads. With very short peak loads the AMD cores seem to even slightly exceed 3200 MHz (checked with `cpuminer` benchmark that scores +170 kH/s on first run to drop to ~160 kH/s with following scores when running continually).



## ASMB10-iKVM BMC

The firmware running on the ASpeed AST2600 BMC is a branded [AMI MegaRAC](https://www.ami.com/megarac/)

## PSUs

2 x 800W redundant [Acbel Polytech R1CA2801A PSUs](https://www.acbel.com/files/product/download/Specification/IPC_PDF/R1CA2301A-B_R1CA2401A-B_R1CA2551K_R1CA2801A-B_R1CA2122A_Specification_01.pdf)

## 

## NICs

  * 2 x onboard Intel I350 GbE interfaces (MAC address OUI: 04:42:1A -> Asus)
  * 2 x SFP+ BCM57840 NetXtreme II 10GbE in OCP 3.0 NIC slot (MAC address OUI: A0:36:BC -> Asus), negotiated link is Gen3 x8
  * virtualized USB Ethernet 

## Memory

4 x 32GB Samsung M393A4K40EB3-CWE 

Up to 4TB RAM

## Fans

7 replaceable fans

## SATA

[    1.828964] ahci 0000:8b:00.0: AHCI 0001.0301 32 slots 4 ports 6 Gbps 0xf impl SATA mode
[    1.828969] ahci 0000:8b:00.0: flags: 64bit ncq sntf ilck pm led clo only pmp fbs pio slum part ems sxs 
[    1.860335] ahci 0000:46:00.0: AHCI 0001.0301 32 slots 8 ports 6 Gbps 0xff impl SATA mode
[    1.860343] ahci 0000:46:00.0: flags: 64bit ncq sntf ilck pm led clo only pmp fbs pio slum part ems sxs 
[    1.902310] ahci 0000:47:00.0: AHCI 0001.0301 32 slots 8 ports 6 Gbps 0xff impl SATA mode
[    1.902316] ahci 0000:47:00.0: flags: 64bit ncq sntf ilck pm led clo only pmp fbs pio slum part ems sxs

ata1-ata20

## Consumption

`echo 1 >/sys/bus/pci/devices/0000\:43\:00.0/remove`
`echo 1 >/sys/bus/pci/devices/0000\:43\:00.1/remove`

55.5W after, 60W

`echo "1" > /sys/bus/pci/rescan`


### Full Speed profile

The fans show up to 27000 rpm and idle consumption of the active PSU is at ~255 W

### Generic profile

Fans 1-5 spin at ~12500 rpm, fans 6-7 are at 10500 rpm, idle consumption of the active PSU is at ~99.5 W

### Custom profile

The active PSU measures ~84.5 W in idle or in other words: 15W less compared to 'Generic profile'. With these custom settings in idle CPU temperature increases by 16°C (from 32°C to 48°C), our Transcend SSDs in the M.2 slots report 11°C more (from 36°C to 47°C) and the U.2 SSDs in the 2.5" bays report an temperature increase of 4-5°C.






idle: 61W, 3000 rpm

cpuminer: 125W @ 160 kH/s, 9500/6000 rpm (BMC readouts: CPU: 72.1W, memory: 16.6W, wall: 130W), 100% CPU utilization

18:15

7-zip: 115W @ 46000 7-ZIP MIPS, 7250/4800 rpm (BMC readouts: CPU: 60.6W, memory: 24.2W, wall: 123W) 90% CPU utilization



Xeon:

idle: 118W, 2700 rpm

cpuminer: 160W @ 114 kH/s, 3550 rpm, 100% CPU utilization

7-zip: 151W @ 29000 7-ZIP MIPS, 3400 rpm, 84% CPU utilization




[set-samsung-speed.sh](https://pastebin.com/FAcbtmNZ) (needs package `pciutils`)

Gen4: 85.3W
Gen3: 80.7W
Gen1: 78.3W





Gen4:

  read: IOPS=129k, BW=504MiB/s (528MB/s)(499MiB/990msec)
    slat (nsec): min=1593, max=147596, avg=1839.88, stdev=484.25
    clat (usec): min=29, max=386, avg=119.88, stdev=35.63
     lat (usec): min=31, max=392, avg=121.77, stdev=35.67
    clat percentiles (usec):
     |  1.00th=[   91],  5.00th=[   95], 10.00th=[   97], 20.00th=[  100],
     | 30.00th=[  103], 40.00th=[  106], 50.00th=[  114], 60.00th=[  123],
     | 70.00th=[  127], 80.00th=[  130], 90.00th=[  135], 95.00th=[  139],
     | 99.00th=[  314], 99.50th=[  330], 99.90th=[  375], 99.95th=[  379],
     | 99.99th=[  383]
   bw (  KiB/s): min=542400, max=542400, per=100.00%, avg=542400.00, stdev= 0.00, samples=1
   iops        : min=135600, max=135600, avg=135600.00, stdev= 0.00, samples=1
  write: IOPS=130k, BW=506MiB/s (531MB/s)(501MiB/990msec); 0 zone resets
    slat (usec): min=3, max=280, avg= 4.56, stdev= 9.24
    clat (nsec): min=1212, max=386293, avg=120141.12, stdev=35861.22
     lat (usec): min=29, max=389, avg=124.76, stdev=36.71
    clat percentiles (usec):
     |  1.00th=[   91],  5.00th=[   95], 10.00th=[   98], 20.00th=[  101],
     | 30.00th=[  103], 40.00th=[  106], 50.00th=[  114], 60.00th=[  123],
     | 70.00th=[  127], 80.00th=[  130], 90.00th=[  135], 95.00th=[  139],
     | 99.00th=[  314], 99.50th=[  326], 99.90th=[  371], 99.95th=[  379],
     | 99.99th=[  383]
   bw (  KiB/s): min=545168, max=545168, per=100.00%, avg=545168.00, stdev= 0.00, samples=1
   iops        : min=136292, max=136292, avg=136292.00, stdev= 0.00, samples=1
  lat (usec)   : 2=0.01%, 50=0.01%, 100=19.04%, 250=78.05%, 500=2.91%
  cpu          : usr=17.90%, sys=78.16%, ctx=303, majf=0, minf=15
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=127647,128353,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=504MiB/s (528MB/s), 504MiB/s-504MiB/s (528MB/s-528MB/s), io=499MiB (523MB), run=990-990msec
  WRITE: bw=506MiB/s (531MB/s), 506MiB/s-506MiB/s (531MB/s-531MB/s), io=501MiB (526MB), run=990-990msec

Gen3: 

  read: IOPS=124k, BW=483MiB/s (507MB/s)(499MiB/1032msec)
    slat (nsec): min=1583, max=141605, avg=1837.29, stdev=464.26
    clat (usec): min=7, max=407, avg=124.87, stdev=42.25
     lat (usec): min=9, max=409, avg=126.76, stdev=42.28
    clat percentiles (usec):
     |  1.00th=[   92],  5.00th=[   96], 10.00th=[   98], 20.00th=[  101],
     | 30.00th=[  103], 40.00th=[  106], 50.00th=[  110], 60.00th=[  129],
     | 70.00th=[  139], 80.00th=[  143], 90.00th=[  147], 95.00th=[  151],
     | 99.00th=[  330], 99.50th=[  338], 99.90th=[  383], 99.95th=[  388],
     | 99.99th=[  404]
   bw (  KiB/s): min=488512, max=499056, per=99.80%, avg=493784.00, stdev=7455.73, samples=2
   iops        : min=122128, max=124764, avg=123446.00, stdev=1863.93, samples=2
  write: IOPS=124k, BW=486MiB/s (509MB/s)(501MiB/1032msec); 0 zone resets
    slat (usec): min=3, max=295, avg= 4.90, stdev=11.53
    clat (nsec): min=1122, max=407342, avg=125341.21, stdev=43300.88
     lat (usec): min=7, max=410, avg=130.30, stdev=44.28
    clat percentiles (usec):
     |  1.00th=[   92],  5.00th=[   96], 10.00th=[   98], 20.00th=[  101],
     | 30.00th=[  104], 40.00th=[  106], 50.00th=[  110], 60.00th=[  128],
     | 70.00th=[  139], 80.00th=[  143], 90.00th=[  147], 95.00th=[  151],
     | 99.00th=[  330], 99.50th=[  338], 99.90th=[  388], 99.95th=[  396],
     | 99.99th=[  404]
   bw (  KiB/s): min=490992, max=500912, per=99.69%, avg=495952.00, stdev=7014.50, samples=2
   iops        : min=122748, max=125228, avg=123988.00, stdev=1753.62, samples=2
  lat (usec)   : 2=0.01%, 10=0.01%, 20=0.01%, 50=0.01%, 100=16.42%
  lat (usec)   : 250=80.17%, 500=3.40%
  cpu          : usr=14.84%, sys=80.60%, ctx=339, majf=0, minf=15
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=127647,128353,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=483MiB/s (507MB/s), 483MiB/s-483MiB/s (507MB/s-507MB/s), io=499MiB (523MB), run=1032-1032msec
  WRITE: bw=486MiB/s (509MB/s), 486MiB/s-486MiB/s (509MB/s-509MB/s), io=501MiB (526MB), run=1032-1032msec


12:52 Gen3 test started, 14:54 switching to Gen4

while true ; do fio --rw=readwrite --name=gen4-128k --bs=128k --direct=1 --name=test --size=100M --numjobs=1 --ioengine=libaio --iodepth=32 ; sync ; sleep 59 ; done


fio --rw=readwrite --refill-buffers --bs=128k --ioengine=libaio --direct=1 --name=test --size=100M --numjobs=4 --group_reporting --iodepth=32 --rate=500k



rate=int	Cap the bandwidth used by this job. The number is in bytes/sec,
		the normal suffix rules apply. You can use rate=500k to limit
		reads and writes to 500k each, or you can specify read and
		writes separately. Using rate=1m,500k would limit reads to
		1MB/sec and writes to 500KB/sec. Capping only reads or
		writes can be done with rate=,500k or rate=500k,. The former
		will only limit writes (to 500KB/sec), the latter will only
		limit reads.
ratemin=int	Tell fio to do whatever it can to maintain at least this
		bandwidth. Failing to meet this requirement, will cause
		the job to exit. The same format as rate is used for
		read vs write separation.
rate_iops=int	Cap the bandwidth to this number of IOPS. Basically the same
		as rate, just specified independently of bandwidth. If the
		job is given a block size range instead of a fixed value,
		the smallest block size is used as the metric. The same format
		as rate is used for read vs write separation.
rate_iops_min=int If fio doesn't meet this rate of IO, it will cause
		the job to exit. The same format as rate is used for read vs
		write separation.





https://www.reddit.com/r/zfs/comments/ljniaw/slow_random_writeread_performance_with_nvme_on_zfs/

recordsize to 4k,8k,16k,64k ... no big impact on the nvme
it is just one drive, so there is not really a vdev config ... i justed xattr, acltype, compression, atime, relatime

https://github.com/openzfs/zfs/issues/10856

https://pve.proxmox.com/wiki/ZFS:_Tips_and_Tricks