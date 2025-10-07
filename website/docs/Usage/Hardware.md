---
title: Resources, hardware
sidebar_position: 4
sidebar_label: Hardware
---

Recommended hardware:
* 32 to 64 GiB of RAM - 16 GiB works but can be challenging depending on client mix
* 4 to 8 CPU cores
* 4TB ["mainstream" SSD](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038) - TLC and DRAM

Generally, 8 GiB of RAM is a very tight fit, with only Nimbus/Geth reported to work. 16 GiB can be a tight fit
depending on client mix, and 32 GiB is recommended. 64 GiB makes it possible to run additional services or
even 3 different client combinations for safety. For 16 GiB RAM, a Nimbus/Nethermind combo works well.

4+ CPU cores are recommended to deal with spikes in processing.

An SSD is required for storage because the node databases are so IOPS-heavy. An Ethereum mainnet node takes ~ 1.1 TiB
of storage initially, as of Jan 2024. The on-disk growth pattern differs between execution clients, see
[resource use](../Usage/ResourceUsage.md).

If you have a 2TB disk, it is expected to last (potentially with execution client pruning) until early 2025.
Remy wrote a [migration guide to 4TB](https://github.com/ethstaker/ethstaker-guides/blob/main/migrating-to-a-larger-disk.md).
Also keep an eye on [EIP-4444](https://eips.ethereum.org/EIPS/eip-4444).

Two home server builds that I like and am happy to recommend are below. Both Intel and AMD support IPMI, which means
they can be managed and power-cycled remotely and need neither a GPU nor monitor. Both support ECC RAM, though the AMD
option as of Sept 2020 was unable to report ECC errors via IPMI, only OS-level reporting worked.

**Intel**

* mITX: 
  * SuperMicro X11SCL-IF(-O) (1 NVMe)
    * Intel i3-9100F or Intel Xeon E-21xx/22xx (i5/7 do not support ECC) - ~ 840 USD with Fractal Node case and NVMe
  * SuperMicro X12STL-IF(-O) (1 NVMe)
    * Intel Xeon E-23xx 
  * SuperMicro X13SCL-IF(-O) (1 NVMe)
    * Intel Xeon E-24xx
* uATX:
  * SuperMicro X11SCL-F(-O) (1 NVMe) or X11SCH-F(-O) (2 NVMe). SCH supports an iGPU
    * Intel i3-9100(F) or Intel Xeon E-21xx/22xx(G) (i5/7 do not support ECC) - ~ 900 USD with Fractal Node case and
NVMe
  * SuperMicro X12STL-F(-O) or X12STH-F(-O) (1 NVMe both). STH supports an iGPU
    * Intel Xeon E-23xx(G)
  * SuperMicro X13SCL-F(-O) (1 NVMe) or X13SCH-F(-O) (2 NVMe)
    * Intel Xeon E-24xx
* Common components:
  * 32 GiB of Micron or Samsung DDR4 UDIMM ECC RAM (unbuffered, **not** registered)
  * [4TB M.2 NVMe SSD](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038)

**AMD**

* mITX:
  * AsRock Rack X570D4I-2T (1 NVMe)
* uATX:
  * AsRock Rack X470D4U or X570D4U (2 NVMe both)
* Common components:
  * AMD Ryzen CPU (Zen2/3), but not APU (APUs do not support ECC)
  * 32 GiB of Micron or Samsung DDR4 UDIMM ECC RAM (unbuffered, **not** registered)
  * [4TB M.2 NVMe SSD](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038)

* uATX Zen 4:
  * AsRock Rack B650D4U
  * AMD Ryzen 7000 CPU (Zen4), but not APU (APUs do not support ECC)
  * 32 GiB of DDR5 UDIMM ECC RAM (unbuffered, **not** registered)
  * [4TB M.2 NVMe SSD](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038)

The SuperMicro eStore is one good source of UDIMM ECC, DDR4 and DDR5 both.

Plus, obviously, a case, PSU, case fans. Pick your own. Well-liked options are Node 304 (mITX) and Node 804 (uATX)
with Seasonic PSUs, but really any quality case that won't cook your components will do. For a small uATX form factor,
consider Silverstone ML04B.

[Joe's hardware roundup](https://github.com/jclapis/rocketpool.github.io/blob/main/src/guides/local/hardware.md) has
additional build ideas.

For the SSD, you'll want decent write endurance and good IOPS. You get better sync and prune performance with
["mainstream" SSDs](https://gist.github.com/yorickdowne/f3a3e79a573bf35767cd002cc977b038), that is, SSDs that use TLC
and have DRAM, not DRAMless or QLC.

You may also consider getting two SSDs and running them in a software mirror (RAID-1) setup, in the OS. That way, data
loss becomes less likely for the chain databases, reducing potential down time because of hardware issues.

Why ECC? This is a personal preference. The cost difference is minimal,
and the potential time savings huge. An Ethereum staking full node does not require
ECC RAM; I maintain it is very nice to have regardless.

With non-ECC RAM, if your RAM goes bad, you will be troubleshooting server
crashes, and potentially spending days with RAM testing tools.

With ECC RAM, if your RAM goes bad, your OS and, if Intel, IPMI, will alert
you to corrected (or uncorrected) RAM errors. You'll want to have set up
email alerts for this. You then buy replacement RAM and schedule downtime.
No RAM troubleshooting required, you will know whether your RAM is functional or has issues
because it will report this to you, and correct single-bit errors.

I am so protective of my time these days that I build even my
home PCs with ECC RAM. You know your own tolerance for troubleshooting
RAM best.
