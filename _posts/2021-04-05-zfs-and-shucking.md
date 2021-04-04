---
layout: post
title:  "ZFS and Shucking HDDs"
---

## Background

My old NAS was finally starting to run out of space, both in terms of GBs,
and in terms of space for additional HDDs.

As my old HDDs were four aging 2 TB HDDs in a RAID5 configuration using `mdadm`,
I decided to move to newer, larger, HDDs.

Using [Disk Prices (UK)](https://diskprices.com/?locale=uk) I found that
I could buy a "WD 14 TB Elements Desktop External Hard Drive - USB 3.0, Black"
for a pretty cheap price, only £210 or £15.00 per TB.

Although this was an external USB HDD, I was planning on "shucking" them, e.g.
removing the outer USB enclosure, and taking out the internal HDD, as the
prices were significantly cheaper, and I felt the savings were worth the risk
of invalidating any waranty the HDDs had.

Additionally, HDDs designed for USB use would probably be quieter than
enterprise-grade HDDs, important for me, since I have to live
with my NAS.

Although lower-end WD drives use shingled magnetic recording (SMR), which
are [basically unusable for RAID/ZFS](https://arstechnica.com/gadgets/2020/06/western-digitals-smr-disks-arent-great-but-theyre-not-garbage/),
this only affects the lower capacity HDDs from Western Digital, not their
14 TB version.

I decided to go for at two parity drives, due to the URE issue: with a
unrecoverable read error (URE) rate of 1 per 10^14 bits (1 per 12.5 TB),
there is a good chance of getting a URE if a parity drive fails. So in the case
of a parity drive failing, we need another parity drive to prevent UREs and data
loss when rebuilding the array.

Finally, I decided to get for OpenZFS for my RAID solution. I previously did not
go for it, mainly because it was previously impossible to expand RAID arrays
in OpenZFS, which is somthing I often did with `mdadm`.

However, it looks like this feature is very close to completion
(see [openzfs/zfs#8853](https://github.com/openzfs/zfs/pull/8853)), so hopefully
it will be ready if I ever need to increase the size of my RAID array.

## Shucking

I bought 3x 14 TB WD Elements External Hard Drives from Amazon UK.

I followed [How to Shuck a WD Elements External Hard Drive](https://www.ifixit.com/Guide/How+to+Shuck+a+WD+Elements+External+Hard+Drive/137646)
from iFixit and shucked the drives, to find:

- 1x WD140EDFZ (WD White)
- 2x UNKNOWN

I didn't bother doing any sort of testing on them,
and they're not WD Red drives (just WD White), but a
[review from schildzilla on reddit](https://old.reddit.com/r/DataHoarder/comments/elels8/wd_my_book_14_tb_shucked_wd140edfz_us7sap140/)
looks pretty positive!

## Moving data process

Unfortunately, my current NAS case only had space for an additional 3.5" HDD.
Because of that, my plan was to:

- Put in one 3.5" HDD and create a OpenZFS RAID-Z2 setup, with two "fake drives",
  so the array is in a degraded state.
- Copy the data over from my old RAID array to the new degraded RAID array.
- Remove the old RAID array, and install my two new 3.5" HDD, then let the
  array rebuild. In case on any errors/issues on the array rebuild, I can still
  reinstall the old RAID array.

### Creating the initial OpenZFS array

Firstly, I installed one shucked 3.5" HDD into the only available space left
in the case of my NAS.

After turning my NAS back on, I ran `sudo fdisk -l` and could not find the
installed drive.

The drive follows the SATA 3.3 specification, where the 3rd power pin disables
the power of the drive. However, my old power supply unit (PSU) predates SATA 3.3,
and was constantly sending 3.3V into this power pin.

I loosely followed the guide from
[Access Random](https://www.instructables.com/How-to-Fix-the-33V-Pin-Issue-in-White-Label-Disks-/),
cutting a piece of tape over this pin.

Since pins 1 and 2 seem to not be used as well (see
[SATA 3.3 pin diagram](https://www.tomshardware.com/news/hdd-sata-power-disable-feature,36146.html)
), I just lazily cut a big piece of tap that covered pins 1 to 3.

After booting back up my NAS, the HDD showed up in the BIOS, and I could see
all 12.75 TiB of it's 14 TB glory after running `sudo fdisk -l`:

```console
me@me:~$ sudo fdisk -l
Disk /dev/sdf: 12.75 TiB, 14000519643136 bytes, 27344764928 sectors
Disk model: WDC WD140EDFZ-11
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
```

Then, I made two sparse files to generate the RAID-Z2 array on:

```bash
mkdir -p ~/openzfs-tmp/ && cd ~/openzfs-tmp/ && truncate --size=14000519643136 fake1.img fake2.img
```

I didn't place them into `/tmp`, since I was worried about losing them when
my NAS rebooted. `truncate` would make the files sparse, so they wouldn't
actually use up 14 TB of space on my small boot drive.

I then created the ZFS pool:

```bash
sudo zpool create -f zfspool raidz2 /dev/sdf ~/openzfs-tmp/fake1.img ~/openzfs-tmp/fake2.img
```

Then immediately offlined the two temp files, so that they wouldn't use up my
precious SSD space:

```bash
sudo zpool offline zfspool ~/openzfs-tmp/fake1.img ~/openzfs-tmp/fake2.img
```

And to confirm, the ZFS pool is now working fine:

```console
me@me:~/openzfs-tmp$ zpool status zfspool
  pool: zfspool
 state: DEGRADED
status: One or more devices has been taken offline by the administrator.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Online the device using 'zpool online' or replace the device with
	'zpool replace'.
  scan: none requested
config:

	NAME                                   STATE     READ WRITE CKSUM
	zfspool                                DEGRADED     0     0     0
	  raidz2-0                             DEGRADED     0     0     0
	    sdf                                ONLINE       0     0     0
	    /home/me/openzfs-tmp/fake1.img  OFFLINE      0     0     0
	    /home/me/openzfs-tmp/fake2.img  OFFLINE      0     0     0

errors: No known data errors
```

### Copying over the data

Firstly, I converted my existing old `mdadm` RAID5 pool into read-only mode.

First, I rand `cat /proc/mdstat`, to find the name of my `mdadm` pool, which
was `md0`:

```console
me@me:~/openzfs-tmp$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] [linear] [multipath] [raid0] [raid1] [raid10] 
md0 : active raid5 sda[4] sdb[0] sdd[3]
      5860147200 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/3] [U_UU]
      bitmap: 10/15 pages [40KB], 65536KB chunk

unused devices: <none>
```

Next, I remounted it in readonly mode:

```bash
sudo mount --options remount,ro /dev/md0 /mnt/md0
```

Finally, I opened up a `tmux` session (so that my copy command wouldn't close if
my SSH connection dropped), and used `rsync` to copy the data over.

```bash
rsync --partial-dir=.rsync-partial --info=progress2 --archive /mnt/md0/ /zfspool/
```
