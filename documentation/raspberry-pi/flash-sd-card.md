# Procedure

First, with the SD card not inserted, enter the command `lsblk -p`

Then, insert the SD card and enter `lsblk -p` again. The new `/dev/sdX` device is the SD card.

To burn the card, enter the following command:

```bash
sudo dd bs=4M if=/home/{HOSTNAME}/Downloads/{IMAGENAME}.img of=/dev/sdc status=progress conv=fsync
```

For the CD card duplicating computer, the command would be following:
```bash
sudo dd bs=4M if=/home/kpm-prod1/Downloads/2020-08-20-raspios-buster-armhf-lite.img of=/dev/sdc status=progress conv=fsync
