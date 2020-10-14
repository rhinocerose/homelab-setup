Mount the `raspbian` image with the following command:
```bash
sudo kpartx -av ~/Desktop/2020-08-20-raspios-buster-armhf-lite.img 
```

You will see a result similar to the following:
```bash
add map loop1p1 (253:0): 0 524288 linear 7:1 8192
add map loop1p2 (253:1): 0 3072000 linear 7:1 532480
```

`loop1p1` is the `/root` folder of the pi, and `loop1p2` is the filesystem.

To mount `/root`, enter the following command:

```bash
sudo mkdir /newroot
sudo mount -o loop /dev/mapper/loop1p1 /newroot/
```

Then, any files can be added to `/newroot`

When done, enter:
```bash
sudo umount /newroot
```

To mount the filesystem, we have to specify the filesystem type:
```bash
sudo mount -t ext4 -o loop /dev/mapper/loop1p2 /newroot/
```
