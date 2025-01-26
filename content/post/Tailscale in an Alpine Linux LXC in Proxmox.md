---
share: true
title: Tailscale in an Alpine Linux LXC in Proxmox
date: 2024-03-03
tags:
  - technology/alpine/tailscale
---
I'll honestly start by saying that a lot of this is generally not supported or recommended, I do it anyway because of the benefits of an LXC container.

I shifted away from this strategy a while back, but for anyone looking up how to do this, here's some documentation.
# Installing Tailscale to Alpine Linux LXC

Install Tailscale
```bash
apk add tailscale
```
Add Tailscale to startup
```bash
rc-update add tailscale
```

Turn off the LXC container and edit the LXC file on the proxmox hypervisor, we will be adding `/dev/net/tun` else Tailscale doesn't want to start.
Use your favourite editor and edit `/etc/pve/lxc/100.conf`, adding the following line **at the end of the file**

```bash
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

`100` being the ID of your LXC container in Proxmox.

Now you can turn the container back on, and if everything went right you will be able to do the following on Alpine Linux 3.21 & 3.20

## Start Tailscale service
```bash
rc-service tailscale start
```

### Alternatives start-ups if those don't work 
```bash
/etc/init.d/tailscale start
```
or
```bash
/usr/sbin/tailscaled --state=/var/lib/tailscale/tailscaled.state --port 41641
```

You can then make it available through SSH or even an [ACL](https://tailscale.com/kb/1018/acls) tag if you have it configured (Which I heavily recommend doing):

```bash
tailscale up --ssh
```
or
```bash
tailscale up --ssh --advertise-tags=tag:
```


## Multiple serve options

You can even "reverse proxy" with Tailscale with their serve option.

{{< box info >}}
**Configuration required**

Make sure you have Tailscale Magic DNS turned on, and HTTPS certificates in the Tailscale settings panel
{{< /box >}}

```bash
tailscale serve --bg https / https://localhost:8443
```

Serve HTTPS from a http localhost

```bash
tailscale serve --bg https 443 http://localhost:80
```

Serve HTTPS from https localhost with untrusted certificate
```bash
tailscale serve --bg https+insecure://localhost:8443
```