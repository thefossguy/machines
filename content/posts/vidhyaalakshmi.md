---

title: "Setup vidhyaalakshmi (Router)"
date: 2022-07-23T08:00:00+05:30
draft: false
toc: true

---

# Debian

## Basic setup

### Set hostname

```bash
sudo hostnamectl set-hostname vidhyalakshmi
```

### Set timezone

```bash
sudo timedatectl set-timezone Asia/Kolkata
```

---

## Networking setup

### If you use PPPoE

This hasn't been tested... RTFM :)

```bash
INTERFACE=
USERNAME=
PASSWORD=

sudo nmcli connection add type pppoe ${INTERFACE} ${USERNAME} ${PASSWORD} 802-3-ethernet.mtu 1452
```

### Static IP

#### IPv4

```bash
NETWORK_CONNECTION=$(nmcli -g name,device connection show | grep 'enp\|eth' | cut -f1 -d":" | head -n 1)

IPV4_DNS="1.1.1.2,1.0.0.2,8.8.8.8,8.8.4.4"
MY_IPV4_ADDR="" # 192.168.1.1/27
MY_IPV4_GTWY=""

sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv4.addresses "${MY_IPV4_ADDR}"
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv4.dns "${IPV4_DNS}"
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv4.gateway "${MY_IPV4_GTWY}"
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv4.ignore-auto-dns yes
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv4.method manual
```

#### IPv6

``` bash
## OPTIONAL
IPV6_DNS="2606:4700:4700:0:0:0:0:1112,2606:4700:4700:0:0:0:0:1002,2001:4860:4860:0:0:0:0:8888,2001:4860:4860:0:0:0:0:8844"
MY_IPV6_ADDR=""
MY_IPV6_GTWY=""

sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv6.addresses "${IPV6_DNS}"
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv6.dns "${MY_IPV6_ADDR}"
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv6.gateway "${MY_IPV6_GTWY}"
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv6.ignore-auto-dns yes
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv6.method manual
```

## Make a bridge

Use **eth1** _NOT eth0_ (or whatever the name ends up being as per freedesktop) as LAN

```bash
```

## Enable IPv4 forwarding

```bash
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
```

## Firewall config

```bash
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --add-service=dns --add-service=dhcp --permanent
sudo firewall-cmd --reload
```

## Reboot

```bash
systemctl reboot
```

---


# OpenBSD

## Installation

```sh
WAN_INTERFACE= #vio0 in VM
LAN_INTERFACE= #vio1 in VM
```

 - Terminal type? [vt220] **`vt220`**
 - Chose your keyboard layout ('?' or 'L' for list) [default] **`us`**
 - System hostname? (short form, e.g. 'foo') **`vidhyalakshmi`**
 - Which network interface do you wish to configure? (or 'done') [abc0] **`${WAN_INTERFACE}`**
 - IPv4 address for `${WAN_INTERFACE}` (or 'autoconf' or 'none') [autoconf] **`autoconf`**
 - IPv6 address for `${WAN_INTERFACE}` (or 'autoconf' or 'none') [autoconf] **`none`**
 - Which network interface do you wish to configure? (or 'done') [done] **`done`**
 - Start sshd(8) by default? [yes] **`yes`**
 - Setup a user? (enter a lower-case loginname, or 'no') [no] **`pratham`**
 - Full name for the user pratham? [pratham] **`Pratham Patel`**
 - Allow root ssh login? (yes, no, prohibit-password) [no] **`no`**
 - What timezone are you in? ('?' for list) [Asia/Kolkata] **`Asia/Kolkata`**
 - Which disk is the root disk? ('?' for details) **`?`**
 - MBR has invalid signature; not showing it. Use (W)hole disk or (E)dit the MBR? [whole] **`W`**
 - Use (A)uto layout, (E)dit auto layout, or create (C)ustom layout? [a] **`A`**
 - Which disk do you wish to initialize? (or 'done') [done] **`done`**
 - Let's install the sets! Location of sets? (disk http nfs or 'done') [http] **`http`**
 - HTTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none] **`none`**
 - HTTP Server? (hostname, list#, 'done' or '?') [cdn.openbsd.org] **`cdn.openbsd.org`**
 - Server directory? [pub/OpenBSD/x.y/riscv64] **`pub/OpenBSD/x.y/riscv64`**
 - Sets: **`-game* -x*`**


## Install updates

```bash
fw_update
pkg_check -Fimv
pkg_add -imUuVv
sysupgrade
```

## Basic setup

### Timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
```

### SSH Config
```bash
echo "ListenAddress 10.0.0.1" >> /etc/ssh/sshd_config
```

### Doas setup

```bash
echo "permit keepenv pratham" > /etc/doas.conf
```

## Reboot

```bash
shutdown -r now
```

## Install new packages

```bash
pkg_add -imUuVv bash bash-completion curl git htop iftop iperf iperf3 pftop vim--no_x11 vnstat wget wireguard-tools
```

## Router setup

Heavily inspired by the official [OpenBSD documentation](https://www.openbsd.org/faq/pf/example1.html)/guide.

### Setup networking

Use the `10.0.0.0/8` subnet for `$WAN_INTERFACE`.

```bash
WAN_INTERFACE= #em0 or vio0 or something else...
LAN_INTERFACE= #em1 or vio1 or something else...
```

```bash
WAN_IF_CONF="inet MY_STATIC_IPv4_HERE 255.255.255.224 NONE
!route add default MY_ISPs_GATEWAY_HERE
inet6 autoconf"

LAN_IF_CONF="inet 10.0.0.1 255.0.0.0 10.0.0.255"
```


```bash
echo 'net.inet.ip.forwarding=1' >> /etc/sysctl.conf
# IPv6 $(echo "net.inet6.ip6.forwarding=1" >> /etc/sysctl.conf)
echo ${WAN_IF_CONF} > /etc/hostname.${WAN_INTERFACE}
echo ${LAN_IF_CONF} > /etc/hostname.${LAN_INTERFACE}
```

### DHCP

```bash
rcctl enable dhcpd
rcctl set dhcpd flags em1 athn0
```

```bash
DHCPD_CONF_CONTENTS="# my main devices go on this subnet; phone, laptop, computer, rpis, etc...
subnet 10.0.0.0 netmask 255.255.255.0 {
	option routers 10.0.0.1;
	option domain-name-servers 10.0.0.1;
	range 10.0.0.10 10.0.0.100;


    # static LAN IP for my primary phone
	host barbet {
		fixed-address 10.0.0.11;
		hardware ethernet 00:00:00:00:00:00;
	}

    # static LAN IP for old phone
	host merlin {
		fixed-address 10.0.0.12;
		hardware ethernet 00:00:00:00:00:00;
	}

    # static LAN IP for old-old phone
	host vince {
		fixed-address 10.0.0.13;
		hardware ethernet 00:00:00:00:00:00;
	}


    # static LAN IP for my MBP (Wi-Fi)
	host vidhata {
		fixed-address 10.0.0.21;
		hardware ethernet 00:00:00:00:00:00;
	}

    # static LAN IP for my Desktop/Workstation
	host harinarayan {
		fixed-address 10.0.0.22;
		hardware ethernet 00:00:00:00:00:00;
	}

    # static LAN IP for my MBP (eth dongle)
	host bramha {
		fixed-address 10.0.0.23;
		hardware ethernet 00:00:00:00:00:00;
	}


    # static LAN IP for my Raspberry Pi 4 Model B 4GB
	host adinath {
		fixed-address 10.0.0.31;
		hardware ethernet 00:00:00:00:00:00;
	}

    # static LAN IP for my Raspberry Pi 4 Model B 8GB
	host balakrishna {
		fixed-address 10.0.0.32;
		hardware ethernet 00:00:00:00:00:00;
	}


    # static LAN IP for my primary WAP
	host rahu {
		fixed-address 10.0.0.90;
		hardware ethernet 00:00:00:00:00:00;
	}
}

# IoT devices go on this subnet; extra WAP, Android set-top box, etc...
subnet 10.0.10.0 netmask 255.255.255.0 {
	option routers 10.0.10.1;
	option domain-name-servers 10.0.10.1;
	range 10.0.10.10 10.0.10.100;


    # static LAN IP for my Android set top box
	host vibhishan {
		fixed-address 10.0.10.11;
		hardware ethernet 00:00:00:00:00:00;
	}


    # static LAN IP for my guest WAP
	host ketu {
		fixed-address 10.0.10.90;
		hardware ethernet 00:00:00:00:00:00;
	}
}
"
```

```bash
echo ${DHCPD_CONF_CONTENTS} > /etc/dhcpd.conf
```

### Firewall

```bash
rcctl enable pf # not really necessary, but OCD
```

```bash
WAN_INTERFACE= #em0 or vio0 or something else...
LAN_INTERFACE= #em1 or vio1 or something else...
```

```bash
PF_CONF_CONTENTS="# set macros for LAN_IF and WAN_IF
LAN_IF = "${LAN_INTERFACE}"
WAN_IF = "${WAN_INTERFACE}"

# network hosts; look at "/etc/dhcpd.conf" for what they are
host_barbet = "10.0.0.11"
host_merlin = "10.0.0.12"
host_vince = "10.0.0.13"

host_vidhata = "10.0.0.21"
host_harinarayan = "10.0.0.22"
host_bramha = "10.0.0.23"

host_adinath = "10.0.0.31"
host_balakrishna = "10.0.0.32"

host_rahu = "10.0.0.90"
host_ketu = "10.0.10.11"

host_vibhishan = "10.0.10.90"

host_pappa = "10.0.10."
host_mummy = "10.0.10."
host_kaki = "10.0.10."
host_kaka = "10.0.10."
host_baa = "10.0.10."
host_dada = "10.0.10."
host_ = "10.0.10."
host_ = "10.0.10."
host_ = "10.0.10."
#host_ = "10.0.0."
#host_ = "10.0.0."
#host_ = "10.0.0."
#host_ = "10.0.0."
#host_ = "10.0.0."
hosts_protected "{" $host_barbet $host_merlin $host_vidhata $host_harinarayan $host_adinath $host_balakrishna $ "}"
hosts_known_guests "{" $host_vince $host_rahu $host_ketu "}"
hosts_totally_isolated "{" $host_vibhishan "}"

# table for blocking IP addresses
# yet to be populated
#table <assholes> const { x.x.x.x, y.y.y.y }
# table for my VLANs
table <vlan_protected_tab> const { 10.0.0.0/8 }
table <vlan_isolated_tab> const { 10.0.10.0/24 }
#table <vlan__tab> const { }

# don't need to route any traffic from loop-back interface
set skip on lo0
# don't respond to the sendor if their packets have been blocked
set block-policy drop

# block everything
block drop all

# passing packets LAN <-> LAN
pass in on $LAN_IF from $LAN_IF:network to any keep state
# allow OpenBSD to connect to the internet (package management, etc)
# pass WAN network to WAN without modification
pass out on $WAN_IF from $WAN_IF:network to any keep state
# pass LAN network OUT to WAN using Network Address Translation
pass out on $WAN_IF from $LAN_IF:network to any nat-to ($WAN_IF) keep state
"
```

```bash
echo ${PF_CONF_CONTENTS} > /etc/pf.conf

pfctl -nf /etc/pf.conf && pfctl -f /etc/pf.conf
```

### DNS

```bash
rcctl enable unbound
```

```bash
```

---

### Monitoring

```bash
systat ifstat
systat netstat
netstat -an
pftop
iftop
vnstatd -d && vnstat
tcpdump
```
