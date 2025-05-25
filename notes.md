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

## Networking

Find other IPv6 nodes on the local network:

```bash
ping6 ff02::1
```

Show statistics for interface:

```bash
ip -s link show eth0
```

## Packages and repository

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

List and install groups of packages:

```bash
dnf group list
dnf group install '<Group_name>'
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

## Timers

To run an `at` command:

```bash
at 21:05 < myscript.sh
at now+5min < myscript.sh
at now+5min -q b < myscript.sh # put it in queue b
atq # list at jobs
at -c  <id> # cat job
atrm <id> # remove at job with id <id>
```

Crontab:

```bash
*/5 * * * * ls # run ls every 5 minutes
5,10,15-20 * * * * ls # run ls at 5 min, 10, 15, 16, 17, 18, 19, 20
```

## Temporary files

Read about them. A little bit complicated

```bash
dnf install systemd-tmpfiles # package that provides this functionality
man tmpfiles.d # your friend
systemd-tmpfiles --clean /path/to/config # to clean
systemd-tmpfiles --create /path/to/config # to create

# after creating a configuration file, run
systemd-tmpfiles --create /path/to/config
```

## Archiving

Automatically choose compressing algorithm by extension:

```bash
tar acf my_gzip_archive.tar.gz /etc
tar axf my_gzip_archive.tar.gz /etc
```

## Remote copy

SFTP seems to be slow but reliable boss of file transfers.
rsync is the fast and reliable boss of file transfers.

## SELinux

In order to fully disable selinux in rhel9:

```bash
# Set the following kernel parameter
selinux=0
```

Settings of SELinux:

```bash
getenforce # gets the active selinux mode
setenforce Enforcing
setenforce Permissive
# change /etc/selinux/config and reboot
semanage fcontext -l # show policies. Might be helpful for those regexes
# see types in /etc/selinux/targeted/contexts/files/file_contexts

semanage fcontext --add --type httpd_sys_content_t '/var(/.*)?' # add another type to /var and anything below it
restorecon -Rv /var # mandatory after an add, delete modify

semanage fcontext --list -C # see modifications made with fcontext
semanage boolean --list -C # see modifications made with boolean

ausearch -m AVC -ts recent # search for recent AVC errors
                           # you can also look in /var/log/messages
sealert -a /var/log/audit/audit.log # search this file for selinux reported errors
#!!! And don't forget the restorecon after doing modifications !!!
```

SELinux add labels to ports:

```bash
semanage port --add --type <type> -p tcp/udp <port_nr>
semanage port -a -t gopher_port_t -p tcp 71
```

## Filesystems and Partitions

Wait for the system to detect the new partition:

```bash
udevadm settle
```

Verify fstab:

```bash
findmnt --verify
```

Swap entry /etc/fstab:

```bash
UUID=77777....  swap    swap    defaults    0 0
```

Test it by running:

```bash
swapon --all
```

Extend filesystem on an xfs lvm:

```bash
xfs_grows /dev/vg1/lv1
```

### Stratis

```bash
dnf install stratis-cli stratisd
systemctl enable --now stratisd

stratis pool create pool1 /dev/vdb # create pool of device(s)
stratis pool list # list pools
stratis pool add-data pool1 /dev/vdc # add another block device to pool

stratis filesystem create pool1 fs1 # create filesystem from pool
stratis filesystem list # list filesystems
stratis filesystem snapshot pool1 fs1 snapshot1 # create a filesystem snapshot

lsblk --output=UUID /dev/stratis/pool1/fs1 # get uuid of stratis filesystem
UUID=c7b57190-8fba-463e-8ec8-29c80703d45e /dir1 xfs defaults,x-systemd.requires=stratisd.service 0 0 # entry in fstab
```

### NFS

Look for available resources:

```bash
showmount --exports <server> # works only on nfs3
mount <server>:/ /mountpoint # works only on nfs4
mount -t nfs -o rw,sync <server>:/<export> /mountpoint # works with all
```

Persistent mount in fstab:

```bash
<server>:/<export>  nfs /<mountpoint>   rw  0   0
```

### Autofs

#### Direct mappings

```bash
# Write the following as if what you want to accomplish is:
#
# $: mount -t nfs -o rw,sync homelab:/media/movies /movies
# $: mount -t nfs -o rw,sync homelab:/media/shows /shows
#
# The only thing you need to adhere to is:
#
# /etc/auto.master.d/<custom_name>.autofs
# /etc/auto.<custom_name>

echo "/-                                /etc/auto.media"         >> /etc/auto.master.d/media.autofs
echo "/movies   -rw,sync,fstype=nfs4    homelab:/media/movies"   >> /etc/auto.media
echo "/shows    -rw,sync,fstype=nfs4    homelab:/media/shows"    >> /etc/auto.media
```

#### Indirect mappings

```bash
# They are similar with direct mapping
#
# The difference between them is that with indirect mapping
# not everything is mounted (thing of a tree of directories)
# only what is specified
#

echo "/media    /etc/auto.movies" >> /etc/auto.master.d/movies.autofs
echo "movies    -rw,sync    homelab:/media/movies" >> /etc/auto.movies

# if you don't want to specify every directory do the following,
# and it will mount what you want to go into and nothing else

echo "/media    /etc/auto.media" >> /etc/auto.master.d/media.autofs
echo "*    -rw,sync    homelab:/media/&" >> /etc/auto.media
```

## Boot process

Get and set default target:

```bash
systemctl get-default
systemctl set-default multi-user.target
```

To do temporary set the target while booting, in grub, add this kernel option:

```bash
systemd.unit=rescue.target
```

### Reset root password

1. Restart the system
2. In grub choose the rescue option
3. Edit before launching:
    - add the kernel parameter rd.break
4. Boot
5. Enter maintenance mode
6. Remount /sysroot with rw:

```bash
mount -o remount,rw /sysroot
```

7. Chroot into /sysroot:

```bash
chroot /sysroot
```

9. Change password

```bash
passwd root
```

10. Trigger selinux to relabel everything:

```bash
touch /.autorelabel
```

11. Exit from everything

### Enable a rooted tty9 for debugging

```bash
systemctl enable debug-shell.service

# add kernel parameter
systemd.debug-shell
```

Sometimes the / (root) will need to be remounted with rw.

## Firewall

```bash
firewall-cmd --get-default-zone
firewall-cmd --set-default-zone=internal

# permanently adds all ip addresses 172.35.23.0 with mask 255.255.255.0 to the public zone and opens ports for mysql
firewall-cmd --permanent --zone=public --add-source=172.35.23.0/24 --add-service=mysql

# permanently add a single address to public zone
firewall-cmd --permanent --zone=public --add-source=172.40.23.111/32

# permanently add port with protocol
firewall-cmd --permanent --zone=public --add-port=82/tcp

firewall-cmd --reload
```

## Podman

```bash
# create a configuration file with the registry
# cat /home/user/.config/containers/registries.conf
# unqualified-search-registries = ['registry.lab.example.com']
#
# [[registry]]
# location = "registry.lab.example.com"
# insecure = true
# blocked = false

podman login <url> --tls-verify=false
podman pull registry.redhat.io/rhel9/rhel-guest-image:9.4 --tls-verify=false
podman images
podman images --all
podman ps
podman run registry.redhat.io/rhel9/rhel-guest-image:9.4 echo "hello"
podman run --name rhel9 registry.redhat.io/rhel9/rhel-guest-image:9.4 echo "hello"
podman run -p 8080:8080 --name rhel9 registry.redhat.io/rhel9/rhel-guest-image:9.4 echo "hello"
podman run -e NAME='Redhat' -p 8080:8080 --name rhel9 registry.redhat.io/rhel9/rhel-guest-image:9.4 printenv NAME
podman run -d -e NAME='Redhat' -p 8080:8080 --name rhel9 registry.redhat.io/rhel9/rhel-guest-image:9.4 printenv NAME
podman run --rm -d -e NAME='Redhat' -p 8080:8080 --name rhel9 registry.redhat.io/rhel9/rhel-guest-image:9.4 printenv NAME
podman container logs <container_name>

# volumes
# the permissions need to match with regards with who owns what

podman exec -it <container_name> <cmd>
podman exec -it db01 grep mysql /etc/passwd
# mysql:x:27:27:MySQL Server:/var/lib/mysql:/sbin/nologin

# launch a command inside a modified user namespace that matches the environment in a container
podman unshare chown 27:27 /home/user/db_data
ls -ld /home/user/db_data
# drwxrwxr-x. 3  100026  100026 18 May  5 14:37 db_data

# it might be useful to look in the configuration part of the image
# to get an idea what is expected
skopeo inspect docker://registry.lab.example.com/rhel9/mariadb-105

# now we can attach the volume
# BE CAREFUL TO AUTOMATICALLY SET THE CORRECT SELINUX LABEL
podman run -d --name db01 \
-e MYSQL_USER=student \
-e MYSQL_PASSWORD=student \
-e MYSQL_DATABASE=dev_data \
-e MYSQL_ROOT_PASSWORD=redhat \
-v /home/user/db_data:/var/lib/mysql:Z \
registry.lab.example.com/rhel8/mariadb-105

ls -Zd /home/user/db_data
# system_u:object_r:container_file_t:s0:c81,c1009 db_data
# the linked volume needs to have container_file_t selinux label

# Create a systemd user service to start the containers at boot
# it is important to generate the file when the container is running
# otherwise it will create an empty container
mkdir -p ~/.config/systemd/user
podman generate systemd --name <container_name> --files # creates /home/user/container-<container_name>.service
podman generate systemd --name <container_name> --files --new # pass new to create containers and delete containers, when booting or shutting down
# move the file in ~/.config/systemd/user
systemctl --user enable --now container-<container_name>.service
loginctl enable-linger # to start services and boot and stop them at shutdown
```
