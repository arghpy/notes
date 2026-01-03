# Samba

```console
$: sudo mount //<IP>/files test -o user=<USER>,defaults,noperm
```

These options **defaults,noperm** ate 1h30' out of my life.
HUGE HUGE thanks to `shrimpwagon` [from the comments](https://unix.stackexchange.com/questions/92168/samba-share-permission-denied-user-writing-file-but-still-shows).
# [Docker](./Docker.md)
## Installation

Install **docker, docker-compose and docker-buildx** on arch linux.
Start and enable **docker.socket** instead of **docker.service** to start docker only on first use, decreasing boot time.
Add yourself to the docker group. Requires logout or `newgrp` command.

# [Git](./Git.md)
## Git submodules

### Adding a submodule

If you want to add a git submodule to an existing repository:
```console
## <URL> should be a git url that anyone could use
git submodule add <URL>
```

The resulting files can be committed separately through a PR.

If you, as a developer of the submodule, would like to also commit
into it you should:
```console
## <PRIVATE_URL> this could be for example the ssh version of the <URL>
git config submodule.DbConnector.url <PRIVATE_URL>
```

### Cloning a project with a submodule

Cloning is done normal.
```console
git clone <URL>
```

Next you will need to run:
```console
git submodule init
```
in order to initialize the local configuration, and:
```console
git submodule update
```
to update all the data and checkout the appropriate commit.

These two previous commands can be combined into one:
```console
git submodule update --init
```

Further more, if the submodules have submodules themselves:
```console
git submodule update --init --recursive
```

All the previous commands in this section can be done in one go:
```console
git clone --recurse-submodules <URL>
```

### Working with a submodule

If you want to get updates from a submodule:
```console
git submodule update --remote
```

Making changes in the submodules is the same as with any other git project.
Checkout a branch and start working on it.

If you want to see the changes that you made, you can do:
```console
git diff --submodule
```

To push the changes, go in the submodule directory and do a `git push`.

If you want to do this push from the main project do:
```console
git push --recurse-submodules=on-demand
```

In order to not specify the recurse-submodules option:
```
git config --global push.recurseSubmodules on-demand
```

If you don't want to specify `--submodule` every time, do:
```console
git config --global diff.submodule log
```
# [Wireguard](./Wireguard.md)
Besides the normal installation with pivpn, you also need to set the following for DNS:
- dnsmasq

## DNSMASQ

You need to disable **systemd-resolved.service** and replace the contents of `/etc/resolv.conf` with:

```text
nameserver 127.0.0.1
```

And create the file `/etc/dnsmasq.d/wg0.conf`:

```text
# Listen on WireGuard interface only
interface=<interface name>       # E.g: wg0
listen-address=<ip_of_wg_server> # E.g: 10.0.0.1
bind-interfaces
no-resolv

# Forward everything else to your LAN router
server=<router_ip>               # E.g: 192.168.1.1
```
