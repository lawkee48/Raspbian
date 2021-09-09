# Raspbian
This repo introduces the procedures to install Raspbian OS for Raspberry Pi. Raspbian is the Raspberry Pi’s most popular operating system, a spin off of the Linux distribution Debian that works well on the Raspberry Pi’s hardware. Raspbian is a competent and versatile operating system that gives your Raspberry Pi all the comforts of a PC: a command line, a browser, and tons of other programs. You can use a Raspberry Pi running Raspbian as a cheap and effective home computer, or you can use it as a springboard and turn your Raspberry Pi into any of countless other functional devices, from wireless access points to retro gaming machines.
## Before install Raspbian OS
- Prepare a microSD card (Min 4GB, Recommend 32GB)

## Option 1: Install Raspberry Pi OS using Raspberry Pi Imager
Raspberry Pi Imager is the quick and easy way to install Raspberry Pi OS and other operating systems to a microSD card, ready to use with your Raspberry Pi.
Donwload Raspberry Pi Imager [here](https://www.raspberrypi.org/software/) and follow the a [46 seconds video](https://youtu.be/ntaXWS8Lk34) which show how to install Raspberry Pi OS using Raspberry Pi Imager.

## Option 2: Manually install an operating system image
### Download Raspbian
Download official images for recommended operating systems from the [Raspberry Pi website downloads page](https://www.raspberrypi.org/software/operating-systems/)
You may need to unzip the downloaded file (.zip) to get the image file (.img) you need to write to the card. The Raspberry Pi OS with desktop image contained in the ZIP archive is over 4GB in size and uses the ZIP64 format. To uncompress the archive, a unzip tool that supports ZIP64 is required. The following zip tools support ZIP64; 7-Zip for Windows, The Unarchiver for macOS, and unzip on Linux.

### 2.1. Installing Images on Linux

[Raspberry Pi Imager](https://www.raspberrypi.org/software/) is typically the easiest option for most users to write images to SD cards, so it is a good place to start. If you're looking for more advanced options on Linux, you can use the standard command line tools below.

- ___NOTE: Using the `dd` tool can overwrite any partition of your machine. If you specify the wrong device when using `dd`, you could delete your primary Linux partition. Please be careful.___

#### Discovering the SD card mountpoint and unmounting it

* Run `lsblk -p` to see which devices are currently connected to your machine.
* If your computer has a slot for SD cards, insert the card. If not, insert the card into an SD card reader, then connect the reader to your computer.
* Run `lsblk -p` again. The new device that has appeared is your SD card (you can also usually tell from the listed device size). The naming of the device will follow the format described in the next paragraph.
* The left column of the results from the `lsblk -p` command gives the device name of your SD card and the names of any partitions on it (usually only one, but there may be several if the card was previously used). It will be listed as something like `/dev/mmcblk0` or `/dev/sdX` (with partition names `/dev/mmcblk0p1` or `/dev/sdX1` respectively), where `X` is a lower-case letter indicating the device (eg. `/dev/sdb1`). The right column shows where the partitions have been mounted (if they haven't been, it will be blank).
* If any partitions on the SD card have been mounted, unmount them all with `umount`, for example `umount /dev/sdX1` (replace `sdX1` with your SD card's device name, and change the number for any other partitions).

#### Copying the image to the SD card

* In a terminal window, write the image to the card with the command below, making sure you replace the input file `if=` argument with the path to your `.img` file, and the `/dev/sdX` in the output file `of=` argument with the correct device name. **This is very important, as you will lose all the data on the hard drive if you provide the wrong device name.** Make sure the device name is the name of the whole SD card as described above, not just a partition. For example: `sdd`, not `sdds1` or `sddp1`; `mmcblk0`, not `mmcblk0p1`.

```
sudo dd if=2021-05-07-raspios-buster-armhf.img of=/dev/sdX bs=4M conv=fsync
```

* Please note that block size set to `4M` will work most of the time. If not,  try `1M`, although this will take considerably longer.

#### Copying a zipped image to the SD card

In Linux it is possible to combine the unzip and SD copying process into one command, which avoids any issues that might occur when the unzipped image is larger than 4GB. This can happen on certain filesystems that do not support files larger than 4GB (e.g. FAT), although it should be noted that most Linux installations do not use FAT and therefore do not have this limitation.

The following command unzips the zip file (replace 2021-05-07-raspios-buster-armhf.zip with the appropriate zip filename), and pipes the output directly to the dd command. This in turn copies it to the SD card, as described in the previous section.

```
unzip -p 2021-05-07-raspios-buster-armhf.zip | sudo dd of=/dev/sdX bs=4M conv=fsync
```

#### Checking the image copy progress

* By default, the `dd` command does not give any information about its progress, so it may appear to have frozen. It can take more some time to finish writing to the card. If your card reader has an LED, it may blink during the write process.
* To see the progress of the copy operation, you can run the dd command with the status option.

```
sudo dd if=2021-05-07-raspios-buster-armhf.img of=/dev/sdX bs=4M conv=fsync status=progress
```

* If you are using an older version of `dd`, the status option may not be available. You may be able to use the `dcfldd` command instead, which will give a progress report showing how much has been written. Another method is to send a USR1 signal to `dd`, which will let it print status information. Find out the PID of `dd` by using `pgrep -l dd` or `ps a | grep dd`. Then use `kill -USR1 PID` to send the USR1 signal to `dd`.

#### Optional: checking whether the image was correctly written to the SD card

* After `dd` has finished copying, you can check what has been written to the SD card by `dd`-ing from the card back to another image on your hard disk, truncating the new image to the same size as the original, and then running `diff` (or `md5sum`) on those two images.
* If the SD card is much larger than the image, you don't want to read back the whole SD card, since it will be mostly empty. So you need to check the number of blocks that were written to the card by the `dd` command. At the end of its run, `dd` will have displayed the number of blocks written as follow:

```
xxx+0 records in
yyy+0 records out
yyyyyyyyyy bytes (yyy kB, yyy KiB) copied, 0.00144744 s, 283 MB/s
```

We need the number `xxx`, which is the block count. We can ignore the `yyy` numbers.

* Copy the SD card content to an image on your hard drive using `dd` again:

```
sudo  dd if=/dev/sdX of=from-sd-card.img bs=4M count=xxx
```

`if` is the input file (i.e. the SD card device), `of` is the output file to which the SD card content is to be copied (called `from-sd-card.img` in this example), and `xxx` is the number of blocks written by the original `dd` operation.

* In case the SD card image is still larger than the original image, truncate the new image to the size of the original image using the following command (replace the input file `reference` argument with the original image name):

```
truncate --reference 2021-05-07-raspios-buster-armhf.img from-sd-card.img
```

* Compare the two images: `diff` should report that the files are identical.

```
diff -s from-sd-card.img 2021-05-07-raspios-buster-armhf.img
```

* Run `sync`. This will ensure that the write cache is flushed and that it is safe to unmount your SD card.
* Remove the SD card from the card reader.

## 2.2. Installing Images on Mac OS

[Raspberry Pi Imager](https://www.raspberrypi.org/software/) is the recommended option for most users to write images to SD cards. However if you do not want to use the Imager you can setill copy an operating system to the card from the command line.

#### Finding the SD Card

Insert the SD card in the slot, or connect the SD card reader with the SD card inside, and type `diskutil list` at the command line. You should see something like this,

```
  $ diskutil list
  /dev/disk0 (internal):
      #:                       TYPE NAME                    SIZE       IDENTIFIER
      0:                       GUID_partition_scheme        500.3 GB   disk0
      1:                       EFI EFI                      314.6 MB   disk0s1
      2:                       Apple_APFS Container disk1   500.0 GB   disk0s2

  /dev/disk1 (synthesized):
      #:                       TYPE NAME                    SIZE       IDENTIFIER
      0:                       APFS Container Scheme -      +500.0 GB   disk1
                               Physical Store disk0s2
      1:                       APFS Volume Macintosh HD     89.6 GB    disk1s1
      2:                       APFS Volume Preboot          47.3 MB    disk1s2
      3:                       APFS Volume Recovery         510.4 MB   disk1s3
      4:                       APFS Volume VM               3.6 GB     disk1s4

  /dev/disk2 (external, physical):
      #:                       TYPE NAME                    SIZE       IDENTIFIER
      0:                       FDisk_partition_scheme       *15.9 GB    disk2
      1:                       Windows_FAT_32 boot          268.4 MB   disk2s1
      2:                       Linux                        15.7 GB    disk2s2
```

Here the SD Card is `/dev/disk2` however your disk and partition list may vary.

#### Copying the Image

___WARNING: Using the `dd` command line tool can overwrite your Mac's operating system if you specify the wrong disk device. If you're not sure about what to do, we recommend you use the [Raspberry Pi Imager](https://www.raspberrypi.org/software/) tool.___

Before copying the image you should unmount the SD Card.

```
diskutil unmountDisk /dev/diskN
```

You can then copy the image,

```
sudo dd bs=1m if=path_of_your_image.img of=/dev/rdiskN; sync
```

replacing `N` with the disk number.

NOTE: You should use `rdisk` (which stands for 'raw disk') instead of `disk`, this speeds up the copying.

This can take a few minutes, depending on the image file size. You can check the progress by pressing `Ctrl+T`. After the `dd` command finishes, you can eject the card:

```
sudo diskutil eject /dev/rdiskN
```

#### Troubleshooting

* If the command reports `dd: /dev/rdiskN: Resource busy`, you need to unmount the volume first `sudo diskutil unmountDisk /dev/diskN`.

* If the command reports `dd: bs: illegal numeric value`, change the block size `bs=1m` to `bs=1M`.

* If the command reports `dd: /dev/rdiskN: Operation not permitted`, go to `System Preferences` \-> `Security & Privacy` \-> `Privacy` \-> `Files and Folders` \-> `Give Removable Volumes access to Terminal`.

* If the command reports `dd: /dev/rdiskN: Permission denied`, the partition table of the SD card is being protected against being overwritten by mac OS. 

Erase the SD card's partition table using this command:

```
sudo diskutil partitionDisk /dev/diskN 1 MBR "Free Space" "%noformat%" 100%
```

That command will also set the permissions on the device to allow writing. 

## 2.3. Installing Images on Windows

[Raspberry Pi Imager](https://www.raspberrypi.org/software/) is our recommended option for most users to write images to SD cards, so it is a good place to start. If you're looking for an alternative on Windows, you can use balenaEtcher, Win32DiskImager or imgFlasher.

#### balenaEtcher

* Download the Windows installer from [balena.io](https://www.balena.io/etcher/)
* Run balenaEtcher and select the unzipped Raspberry Pi OS image file
* Select the SD card drive
* Finally, click *Burn* to write the Raspberry Pi OS image to the SD card
* You'll see a progress bar. Once complete, the utility will automatically unmount the SD card so it's safe to remove it from your computer.

#### Win32DiskImager

* Insert the SD card into your SD card reader. You can use the SD card slot if you have one, or an SD adapter in a USB port. Note the drive letter assigned to the SD card. You can see the drive letter in the left hand column of Windows Explorer, for example *G:*
* Download the Win32DiskImager utility from the [Sourceforge Project page](http://sourceforge.net/projects/win32diskimager/) as an installer file, and run it to install the software.
* Run the `Win32DiskImager` utility from your desktop or menu.
* Select the image file you extracted earlier.
* In the device box, select the drive letter of the SD card. Be careful to select the correct drive: if you choose the wrong drive you could destroy the data on your computer's hard disk! If you are using an SD card slot in your computer, and can't see the drive in the Win32DiskImager window, try using an external SD adapter.
* Click 'Write' and wait for the write to complete.
* Exit the imager and eject the SD card.

#### Upswift imgFlasher

* Download portable Windows version from [upswift.io](https://www.upswift.io/imgflasher/)
* Run imgFlasher and choose an image or zip file
* Choose SD card or USB drive
* Click on 'Flash'
* Wait until the flash is completed.

## Change Search Engine in Chromium
Open Chromium browser --> setting --> Change the option under **Search engine**

## Install Python, Python3
Most distributions of Linux come with Python and Python 3 (also thier pip) already installed (Raspbian: Python 2.7.16 & Python 3.7.3), but they might not have IDLE, the default IDE (interactive development environment), installed.
```
sudo apt update
sudo apt install python python3 python-pip python3-pip idle3 idle-python2.7
```

## (Optional) Change pip source
Recommand:
```
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
or temporarily:
```
pip3 install -i http://pypi.douban.com/simple/ <package_name>
```
Sometimes there may be network problem causing a timeout, then we can set a timeout attribute to limit it
```
pip3 --default-timeout=100 install <package_name>
```
Other source list from Mainland China:
- 阿里雲 http://mirrors.aliyun.com/pypi/simple/
- 中國科技大學 https://pypi.mirrors.ustc.edu.cn/simple/
- 豆瓣 http://pypi.douban.com/simple/
- Python官方 https://pypi.python.org/simple/
- v2ex http://pypi.v2ex.com/simple/
- 中國科學院 http://pypi.mirrors.opencas.cn/simple/
- 清華大學 https://pypi.tuna.tsinghua.edu.cn/simple/

## Reference
- How to install Raspbian on the Raspberry Pi. https://thepi.io/how-to-install-raspbian-on-the-raspberry-pi/
- Raspberry Pi Documentation. Getting Started. https://www.raspberrypi.org/documentation/computers/getting-started.html
- Raspberry Pi OS. https://www.raspberrypi.org/software/
