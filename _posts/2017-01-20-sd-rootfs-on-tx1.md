---
layout: post
comments: true
title: "Booting Rootfs off SD Card on Jetson TX1"
excerpt: "I build my TX1 software development environment on large SD Card. This approach has a number of benefits. I share how I build the SD Card image here."
date: 2017-01-20
category: "TX1"
tags: installation
---

NVIDIA's Jetson TX1 Evaluation Board has an eMMC of 16GB. This eMMC storage space is mostly used as Linux rootfs (/), besides BCT and U-Boot, etc. This rootfs (APP) partition gets filled up quickly after a complete installation of the [JetPack](https://developer.nvidia.com/embedded/jetpack). So I prefer not to develop off the eMMC rootfs partition directly.

One very nice design of NVIDIA's Jetson TX1 platform is its U-Boot would look for rootfs on SD Card (/dev/mmcblk1p1) first, followed by eMMC (/dev/mmcblk0p1) and then USB. That is, if we put a working rootfs on a SD Card, we can boot TX1 off the SD Card by just plugging it in before TX1 boots. On the other hand we can fall back to booting from eMMC, on the fly, by just removing the SD Card before TX1 boots.

So I'd build my software development environment on a larger SD Card, and let TX1 boot rootfs off it. This way, my TX1 software development process would not be limited to 16GB of rootfs. One of my preferred SD Card for this purpose is Transcend 64GB SDXC UHS-I U3 95/60 Mb/s, which strikes a good balance between its Read/Write speed and price. I have a few of these SD Cards so that I can switch among different software environments, say 1 for demo and the other for development, quickly.

[http://24h.pchome.com.tw/prod/DGAG0N-A9005IG6Y](http://24h.pchome.com.tw/prod/DGAG0N-A9005IG6Y)
[https://www.amazon.com/Transcend-Speed-UHS-3-Memory-TS64GSDU3/dp/B00J3KA814/ref=sr_1_1?ie=UTF8&qid=1487079911&sr=8-1&keywords=transcend+64GB+SDXC+UHS-I+U3](https://www.amazon.com/Transcend-Speed-UHS-3-Memory-TS64GSDU3/dp/B00J3KA814/ref=sr_1_1?ie=UTF8&qid=1487079911&sr=8-1&keywords=transcend+64GB+SDXC+UHS-I+U3)
![Transcend 64GB SDXC UHS-I U3](http://a.ecimg.tw/items/DGAG0NA9005IG6Y/000001_1478355499.jpg)

Here is my SOP of building rootfs on the SD Card.

1. Do a complete installation of JetPack, say JetPack-2.3.1, on Jetson TX1. This would install everything fresh on the eMMC rootfs partition, including CUDA Toolkit, OpenCVForTegra, VisionWorks, TensorRT and Multimedia API.
2. (Optional but preferred) Update all apt packages to the latest version by `sudo apt-get upgrade`
3. Insert the SD Card onto Jetson TX1. The partition on the SD Card might be mounted automatically. I would then do: (a) un-mount and format the SD Card partition, (b) mount the SD Card partition and copy everything from eMMC rootfs to it, (c) modify boot/extlinux/extlinux.conf so that Linux would use the SD Card partition as root, (d) modify lib/init/fstab.
4. Reboot Jetson TX1 with the SD Card plugged-in. If everything goes as expected, Jetson TX1 would now be using /dev/mmcblk1p1 (SD Card) as rootfs. This could be verified with either `df -h`, `lsblk` or just `mount`.

Detailed commands for step 3 above are: 

```shell_session
 ### Umount and format /dev/mmcblk1p1. This would clean up the partition.
 $ sudo umount /dev/mmcblk1p1
 $ sudo mkfs.ext4 /dev/mmcblk1p1

 ### Mount the freshly formatted /dev/mmcblk1p1. The UUID would differ for
 ### your SD Card partition.
 $ sudo mount /dev/mmc1blk1p1 /media/ubuntu/9ae7c5ba-e812-445b-bfce-20f9d2ac78b6

 ### Copy all files from rootfs to /dev/mmcblk1p1, while excluding the
 ### directory where /dev/mmcblk1p1 is mounted. This step might take 10~20
 ### minutes.
 $ sudo rsync -avxHAX --progress / /media/ubuntu/9ae7c5ba-e812-445b-bfce-20f9d2ac78b6/

 ### Specify "root=/dev/mmcblk1p1" in extlinux.conf
 $ sudo vim /media/ubuntu/9ae7c5ba-e812-445b-bfce-20f9d2ac78b6/boot/extlinux/extlinux.conf

 ### Specify "/dev/mmcblk1p1" as the mount device for root (/)
 $ sudo vim /media/ubuntu/9ae7c5ba-e812-445b-bfce-20f9d2ac78b6/lib/init/fstab
 $ sync
```
Once done, I almost always use the SD Card for software development afterwards. For example, I'd install Torch7, OpenCV-3.x and other stuffs on the SD Card partition only. This way I don't need to worry about eMMC running out of space.

One caveat worth mentioning is that there could be compatibility issue if L4T version on the SD Card partition does not match that on the eMMC rootfs partition. If you encounter such a problem and thus TX1 could not boot successfully off the SD Card , I'd suggest you to re-make the SD Card partition by following those steps outlined above.

Lastly, here's my list of benefits about using SD Cards as the software development environment.

* Since we keep a fresh JetPack environment on the eMMC, we can always boot off eMMC and do benchmarking (or debugging). It's much easier to determine whether a problem is cause by NVIDIA or ourselves this way.
* As mentioned earlier, we can switch between different software (or demo) environments quickly by swapping SD Cards.
* If the SD Card partition is corrupted, we can just boot from eMMC and re-make the SD Card partition. I found this quite useful when I was developing device drivers or modifying Linux kernel for TX1.
* I think the same approach also works for Jetson TK1.

