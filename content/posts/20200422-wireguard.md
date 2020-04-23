---
url: /2020/wireguard/
title: Wireguard and Linux network namespaces - perfect fit
modified: 2020-04-22T20:35:00+03:00
highlight: true
---
# Wireguard and Linux network namespaces - perfect fit

I'm using two browsers usually - Firefox (switched to IceCat recently) and
Chromium.

Firefox/IceCat is my workhorse, which is configured to route all
traffic through SOCKS5 proxy ([Linode][linode] instance actually).

I would definitely stay solely with it but some sites don't like requests
coming from VPS servers so I have to use Chromium for such cases (and it always
run in private mode to keep it clean).

SOCKS5 proxy works pretty good for me but I was curious if it's possible to
use [wireguard][wireguard] for my usecase - to route Firefox/IceCat traffic
through VPN keeping all other traffic as is.

For domain name resolution I'm using locally installed [Unbound][unbound]
server which is bound to localhost. My `/etc/resolv.conf` file looks like this:

```
nameserver 127.0.0.1
```
It is configured to forward requests to [Quad9][quad9] resolver (no Google, no
Cloudfront, right) but I'm planning to turn forwarding off and just use
[Unbound][unbound] on its own.

There are numerous posts in the Internet covering [wireguard][wireguard]
configuration so I'll just show my server/client configuration with some notes.

For some reason I've missed the fact that `wg-quick` helper uses advanced
version of `wg` configuration file syntax which means `wg-quick` configuration
file can't be used with `wg`. I've lost some time while battling this issue.

Summary of the settings:

- `10.10.10.0/24` - VPN network
- `10.10.10.1/32` - VPN server address
- `10.10.10.10/32` - VPN client address
- `55000` - VPN server port wireguard will be listening on
- VPN server will route all traffic coming from client to the outside world


## Server

My server configuration file looks like this:

```ini
# /etc/wireguard/wg0.conf on server

[Interface]
PrivateKey = <private key of the server>
ListenPort = 55000

[Peer]
PublicKey = <public key of the client>
PresharedKey = <preshared key>
AllowedIPs = 10.10.10.10/32
```

I use [Alpine Linux][alpine] on the server for my experiments with wireguard
and my `wg0` interface is defined like this:

```
# excerpt from /etc/network/interfaces

auto wg0
iface wg0 inet static
  address 10.10.10.1
  netmask 255.255.255.0
  pre-up ip link add dev wg0 type wireguard
  pre-up wg setconf wg0 /etc/wireguard/wg0.conf
  pre-down ip link delete dev wg0
```

One more player here is firewall (`iptables` in my case). It must be configured
to allow traffic coming from `wg0` interface to be forwarded to outside world.

Something like this should help you to get started:

```shell
$ sudo iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
$ sudo iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
$ sudo iptables -t nat -A POSTROUTING -o eth0 -s 10.10.10.0/24 -j MASQUERADE
```

Don't forget to make these rules permanent and allow forwarding on the server
using `sysctl`:

```
net.ipv4.ip_forward=1
```

That's it for the server.


## Client

Client configuration file looks like this:

```ini
# /etc/wireguard/wg0.conf on the client

[Interface]
PrivateKey = <private key of the client>

[Peer]
PublicKey = <public key of the server>
PresharedKey = <preshared key>
Endpoint = <server public ip>:55000
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

I've faced another issue while configuring the client - something absolutely
unexpected when pinging any host, even VPN server endpoint:

```shell
$ ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
From 10.10.10.10 icmp_seq=1 Destination Host Unreachable
ping: sendmsg: Required key not available
From 10.10.10.10 icmp_seq=2 Destination Host Unreachable
ping: sendmsg: Required key not available
From 10.10.10.10 icmp_seq=3 Destination Host Unreachable
ping: sendmsg: Required key not available
^C
--- 10.10.10.1 ping statistics ---
3 packets transmitted, 0 received, +3 errors, 100% packet loss, time 2028ms
```

The same for pinging Google DNS:

```shell
$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
From 10.10.10.10 icmp_seq=1 Destination Host Unreachable
ping: sendmsg: Required key not available
From 10.10.10.10 icmp_seq=2 Destination Host Unreachable
ping: sendmsg: Required key not available
From 10.10.10.10 icmp_seq=3 Destination Host Unreachable
ping: sendmsg: Required key not available
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 0 received, +3 errors, 100% packet loss, time 2037ms
```

The reason for this was `AllowedIPs` misconfiguration:

```ini
# This is wrong setting
AllowedIPs = 0.0.0.0
```

I've somehow lost CIDR part from the address which means address will look like
`0.0.0.0/32` which is absolutely wrong.

Pay attention to `wg` output, to `allowed ips` value especially:

```shell
$ sudo wg
interface: wg0
  public key: <client public key>
  private key: (hidden)
  listening port: 52312

peer: <server public key>
  preshared key: (hidden)
  endpoint: <server ip address>:55000
  allowed ips: 0.0.0.0/32
  latest handshake: 1 minute, 12 seconds ago
  transfer: 644 B received, 2.23 KiB sent
  persistent keepalive: every 25 seconds
```

So, correct value for `AllowedIPs` should look like this:

```ini
# This is correct setting
AllowedIPs = 0.0.0.0/0
```

And now we've come to the most interesting part of the show - to client-side
network configuration.

The solution for my usecase was [here][wireguard-netns-sample-script] - simple
script by (probably) [wireguard][wireguard] author. Thank you, Jason!

After some experiments I've taken an opposite approach from the original -
prepared `wg0` interface is moved to `vpn` namespace and is configured to be
the default route within this namespace.

We can check what interfaces our new `vpn` namespace has and how routing table
looks like:

```shell
$ vpn up
...
$ vpn exec ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
10: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/none
...
$ vpn exec ip route
default dev wg0 scope link
10.10.10.0/24 dev wg0 proto kernel scope link src 10.10.10.10
```

Pretty good, right?

One more minor issue was the lack of domain name resolution for programs
running within `vpn` namespace. I was not expecting this so it took some time
to understand the reason.

Citing [ip-netns][ip-netns-man] manual:

> A network namespace is logically another copy of the network stack, with
> its own routes, firewall rules, and network devices.

This means nobody is listening on `127.0.0.1:53` within `vpn` namespace. We can
easily solve this issue by starting another copy of [unbound][unbound] within
`vpn` namespace (I've decided to use separate copy of `unbound.conf` for this
named `unbound.vpn.conf`, YMMV).

Here is the script I'm using to manage VPN:

```bash
#!/usr/bin/env bash

set -ex

[[ $UID != 0 ]] && exec sudo -E "$(readlink -f "$0")" "$@"

NS="vpn"
WGIF="wg0"
WGCONF="/etc/wireguard/$WGIF.conf"
WGADDRIPV4="10.10.10.10/24"
UNBOUNDCONF="/etc/unbound/unbound.$NS.conf"
UNBOUNDPID="/run/unbound.$NS.pid"

up() {
    ip netns add $NS
    ip link add $WGIF type wireguard
    wg setconf $WGIF $WGCONF
    ip link set $WGIF netns $NS
    ip -n $NS addr add $WGADDRIPV4 dev $WGIF
    ip -n $NS link set lo up
    ip -n $NS link set $WGIF up
    ip -n $NS route add default dev $WGIF
    ip netns exec $NS unbound -c $UNBOUNDCONF
}

down() {
    UPID=`cat $UNBOUNDPID`
    kill $UPID || true
    ip -n $NS link set $WGIF down
    ip -n $NS link del $WGIF
    ip netns del $NS
}

status() {
    exec ip netns exec $NS wg
}

execi() {
    exec ip netns exec $NS sudo -E -u \#${SUDO_UID:-$(id -u)} -g \#${SUDO_GID:-$(id -g)} -- "$@"
}

command="$1"
shift

case "$command" in
    up) up "$@" ;;
    down) down "$@" ;;
    exec) execi "$@" ;;
    status) status "$@" ;;
    *) echo "Usage: $0 up|down|exec" >&2; exit 1 ;;
esac
```

Now I can start Firefox/IceCat within `vpn` namespace and be sure all traffic
is routed through VPN:

```shell
$ vpn exec icecat
```

I'm using `netctl` to manage my network interfaces and the plan is to use
[netctl hooks][netctl-hooks] to bring VPN interface up or down as needed.

[wireguard][wireguard] looks quite good (one more time - thank you, Jason A.
Donenfeld!) and I'm planning to use it for some time to decide if I can switch
from SOCKS5 proxy to VPN.


[linode]: https://www.linode.com
[wireguard]: https://www.wireguard.com
[unbound]: https://nlnetlabs.nl/projects/unbound/about/
[quad9]: https://www.quad9.net
[alpine]: https://www.alpinelinux.org
[wireguard-netns-sample-script]: https://www.wireguard.com/netns/#sample-script
[ip-netns-man]: http://manpages.ubuntu.com/manpages/trusty/man8/ip-netns.8.html
[netctl-hooks]: https://wiki.archlinux.org/index.php/Netctl#Using_hooks
