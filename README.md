# how to resize disk on freeBSD on Azure

This example is based on freeBSD 12 on Microsoft Azure

information adopted from these sources

[https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/disks-growing.html]
[https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/geom-glabel.html]
[https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/adding-swap-space.html]
[https://www.vultr.com/docs/how-to-resize-a-disk-in-freebsd]


## WARNING !

Do not reboot your system until you relabel resized disk.
If you do, you will have to use Azure serial console to relabel disk. See step 3.



## steps

1. delete swap partition at the end of disk
2. extend ufs partiotion
3. relabel partition
4. grow labeled partition
5. add and enable swapfile


## 1. delete swap partition at the end of disk

```
# gpart show da0
=>      34  83886013  ada0  GPT  (48G) [CORRUPT]
        34       128     1  freebsd-boot  (64k)
       162  79691648     2  freebsd-ufs  (38G)
  79691810   4194236     3  freebsd-swap  (2G)
  83886046         1        - free -  (10G)
```


## 2. extend ufs partiotion

I dont follow steps as decribed in FreeBSD documentation [17.3. Resizing and Growing Disks](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/disks-growing.html) as i do not want swap partition, and i use swapfile, described below.

disable swap
```
# swapoff /dev/da0p3
```

delete swap partition
```
# gpart delete -i 3 da0
da0p3 deleted
# gpart show ada0
=>       34  102399933  ada0  GPT  (48G)
         34        128     1  freebsd-boot  (64k)
        162   79691648     2  freebsd-ufs  (38G)
   79691810   22708157        - free -  (10G)
```

resize partition to take all space
```
# gpart resize -i 2 da0
da0p2 resized
# gpart show da0
=>       34  102399933  ada0  GPT  (48G)
         34        128     1  freebsd-boot  (64k)
        162   98566144     2  freebsd-ufs  (48G)
```

## 3. relabel partition

Follow steps as decribed in FreeBSD documentation
[18.7. Labeling Disk Devices]([https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/geom-glabel.html])

```
# glabel label rootfs /dev/da0p2
```


## 4. grow labeled partition

```
# growfs /dev/label/rootfs
Growing root partition to fill device

 53224896, 54248448, 55272000, 56295552, 57319104, 58342656, 59366208, 60389760, 61413312,
 62436864, 63460416, 64483968, 65507520, 66531072, 67554624, 68578176, 69601728, 70625280,
 71648832, 72672384, 73695936, 74719488, 75743040, 76766592, 77790144, 78813696, 79837248,
 80860800, 81884352, 82907904

```

## 5. add and enable swapfile

simply follow steps on FreeBSD documentation [11.2. Creating a Swap File on FreeBSD 10.X and Later](https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/adding-swap-space.html#swapfile-10-and-later)


Create 2GB swap file:
```
# dd if=/dev/zero of=/swapfile bs=1m count=2000
```
Set the proper permissions on the new file:
```
# chmod 0600 /usr/swapfile
```
Inform the system about the swap file by adding a line to /etc/fstab:
```
md99	none	swap	sw,file=/swapfile,late	0	0
```
*The md(4) device md99 is used, leaving lower device numbers available for interactive use.*

Swap space will be added on system startup. To add swap space immediately, use swapon(8):
```
# swapon -aL
```



