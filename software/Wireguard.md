Besides the normal installation with pivpn, you also need to set the following for DNS:
- dnsmasq

# DNSMASQ

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
