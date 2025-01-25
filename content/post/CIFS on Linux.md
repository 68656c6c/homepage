---
share: true
title: CIFS on Linux
date: 2025-01-25
filename: guide/20230503-cifs_on_linux
tags:
  - technology/cifs
  - technology/smb
---

```bash
sudo apt install cifs-utils
```
utf8 seems to not be available in ARM Ubuntu Server 22.04 systems, you can check that.
```bash
ls /lib/modules/$(uname -r)/kernel/fs/nls/
```

To download the rest of the charsets.
```bash
sudo apt install linux-modules-extra-$(uname -r)
```

To mount SMB3 with seal (encryption)
```bash
sudo mount.smb3 -o seal,credentials=/etc/cifs-credentials.txt //-sub2.your-storagebox.de/-sub2 /mnt/containers
```

/etc/fstab
```bash
//-sub2.your-storagebox.de/-sub2 /mnt/containers cifs vers=3.1.1,seal,iocharset=utf8,rw,credentials=/etc/cifs-credentials.txt,uid=1001,gid=1001,file_mode=0660,dir_mode=0770 0 0
```

chmod 0600
```bash
sudo vim /etc/cifs-credentials.txt
```

```bash
username=
password=
```