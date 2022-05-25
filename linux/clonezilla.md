# docs: linux/clonezilla
#linux #clonezilla
## create live usb
> Official docs: https://clonezilla.org/liveusb.php
> Download clonezilla stable (.zip):  https://clonezilla.org/downloads.php

> **Important**: change disk and partition, this example uses `/dev/sdb1`

1. Format usb device with FAT32
2. Create `/tmp/usb`
```bash
mkdir -p /tmp/usb
```
3. Mount usb device
```bash
sudo mount /dev/sdb1 /tmp/usb
```
4. Unzip Clonezilla
```bash
unzip ~/Downloads/clonezilla-live-2.6.3-7-amd64.zip -d /tmp/usb
```
5. Make usb bootable
```bash
cd /tmp/usb/utils/linux
bash makeboot.sh /dev/sdb1
```
6. Activate usb boot in BIOS
7. Boot from usb device, have a external HD by the side.

## Clone Notebook
### Save disk image
1. Boot the machine via Clonezilla live
2. The boot menu of Clonezilla live
3. Here we choose 800x600 mode, after pressing Enter, you will see Debian Linux booting process
4. Choose language
5. Choose keyboard layout
6. Choose `Start Clonezilla`
7. Choose `device-image` option
8. Choose `local_dev` option to assign the external disk as the image home
9. Select external disk as image repository, then choose `savedisk` option
10. Input image name and select source disk (computer disk)
11. Clonezilla is saving disk image (computer disk) to the partition of the external disk

### Restore disk image
1. Boot the machine via Clonezilla live
2. The boot menu of Clonezilla live
3. Here we choose 800x600 mode, after pressing Enter, you will see Debian Linux booting process
4. Choose language
5. Choose keyboard layout
6. Choose `Start Clonezilla`
7. Choose `device-image` option
8. Choose `local_dev` option to assign the external disk as the image home
9. Select external disk as image repository, then choose `restoredisk` option
10. Select image name and destination disk (new computer disk)
11. Clonezilla is restoring disk image on external disk to new computer disk