+++
date = "2015-01-25T00:00:00+00:00"
title = "Natively boot Windows with a virtual HDD"
tags = ["Windows"]
aliases = ["2015/natively-boot-windows-10-technical-preview-with-a-virtual-hd"]
+++

A few days ago [Microsoft released a new build for Windows 10](https://insider.windows.com) and I wanted to give it a try. However, I didn't want to resize my partitions or overwrite my Windows 8.1 partition.

>Well, then install it in a VM!

Then I wouldn't have access to all of my RAM and CPU, so that wasn't a solution either. However, I found the ideal solution.

Apparently you can __install Windows in a virtual hard drive__. Let me guide you through:

1. Create a new VHD-file and remember the path to it. You can use _Disk Management_ (run `diskmgmt.msc`) and click _Action_ and _Create VHD_. I made it about 70 GB big with a dynamic size.
2. Boot into the Windows installation (use [Rufus](https://rufus.akeo.ie/) or something similar to create a bootable USB stick).
3. Select _Custom: Install Windows only (Advanced)_, not an _Upgrade_.
4. Now you have to select the partition on which you want to install Windows. Press _SHIFT+F10_ to open a command prompt instead.
5. Enter `diskpart` to open the DiskPart utility.
6. In DiskPart enter `select vdisk file=C:\Windows10.vhd` with the correct path to your VHD to select the VHD.
7. Now enter `attach vdisk` and close the command prompt.
8. If you now press _Refresh_ you will see unallocated space on a newly added disk station, that is your virtual hard drive. Select it.
9. You can now continue installing Windows as usual.

I was surprised to find out about this possibility. This way you can try the new Windows _in full_ without compromising your production environment.
