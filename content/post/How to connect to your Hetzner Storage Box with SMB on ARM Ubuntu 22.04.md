---
share: true
title: How to connect to your Hetzner Storage Box with SMB on ARM Ubuntu 22.04
date: 2023-05-03
tags:
  - technology/cifs
  - technology/smb
---

First you install the CIFS utils.
```bash
sudo apt install cifs-utils
```

utf8 seems to not be available in ARM Ubuntu Server 22.04 systems, you can check that.
```bash
ls /lib/modules/$(uname -r)/kernel/fs/nls/
```

To download the rest of the character sets.
```bash
sudo apt install linux-modules-extra-$(uname -r)
```

To mount SMB3 with seal (encryption) make sure you create a subuser in your storagebox and assign SMB, you can then create the file `/etc/cifs-credentials.txt` with the following text.

```bash
username=u000000-sub1
password=
```

Make sure you put the permissions on `0600` by running the following command:

```bash
sudo chmod 0600 /etc/cifs-credentials.txt
```

Then try to mount it with `sudo`, with the following command:
```bash
sudo mount.smb3 -o seal,credentials=/etc/cifs-credentials.txt //u000000-sub1.your-storagebox.de/u000000-sub1 /mnt/hetznerbox
```

If you receive no error messages, you go to `/mnt/hetznerbox` and create a file or folder, here we create a file:
`touch /mnt/hetznerbox/test`

Try to unmount it after creating the file, and see if the file is no longer there:
```bash
sudo unmount /mnt/hetznerbox
```

And check for the file:
```bash
ls /mnt/hetznerbox
```

If the file is no longer there, and after running the mount again, and `ls` in the folder and the file has reappeared, you can confirm that your mount is working properly.

You can also make sure by activating `SFTP` in your Storage Box with the same credentials so you can verify with `WinSCP` or `Nautilus`

After all that is done, you can edit the `/etc/fstab` file so you can make sure it mounts on boot.
## /etc/fstab
```bash
//u000000-sub1.your-storagebox.de/u000000-sub1 /mnt/containers cifs vers=3.1.1,seal,iocharset=utf8,rw,credentials=/etc/cifs-credentials.txt,uid=1001,gid=1001,file_mode=0660,dir_mode=0770 0 0
```
