---

title: "Setup vidhyaalakshmi (Router)"
date: 2022-07-23T08:00:00+05:30
draft: false
toc: true

---

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
sudo nmcli connection modify "${NETWORK_CONNECTION}" ipv4.gateway "${MY_IPV4_GTWY"
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



