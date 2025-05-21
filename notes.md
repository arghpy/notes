# Samba

```console
$: sudo mount //<IP>/files test -o user=<USER>,defaults,noperm
```

These options **defaults,noperm** ate 1h30' out of my life.
HUGE HUGE thanks to `shrimpwagon` [from the comments](https://unix.stackexchange.com/questions/92168/samba-share-permission-denied-user-writing-file-but-still-shows).

# Docker

## Installation

Install **docker, docker-compose and docker-buildx** on arch linux.
Start and enable **docker.socket** instead of **docker.service** to start docker only on first use, decreasing boot time.
Add yourself to the docker group. Requires logout or `newgrp` command.

# RHCSA

Find other IPv6 nodes on the local network:

```bash
ping6 ff02::1
```

Show statistics for interface:

```bash
ip -s link show eth0
```

Extract local package:

```bash
rpm2cpio <package> | cpio -idv
```

List files in local package:

```bash
rpm2cpio <package> | cpio -tv
```

List and enable rhel repository:

```bash
dnf repolist all
dnf config-manager --enable <repo>
```

Import gpgkey:

```bash
rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9
```

To create a diagnostics report:

```bash
dnf install sos
sos report
sos clean </path/to/report> # cleans the personal information from report
```
