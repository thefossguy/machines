---

title: "Setup vidhyaa (Debian: Router)"
date: 2022-07-23T08:00:00+05:30
draft: false
toc: true

---


### Networking setup

#### DNS setup

```bash
NETWORK_CONNECTION=$(nmcli -g name,device connection show | grep 'enp\|eth' | cut -f1 -d":" | head -n 1)

IPV4_DNS="1.1.1.2,1.0.0.2,8.8.8.8,8.8.4.4"
MY_IPV4_ADDR="" # 192.168.1.1/27
MY_IPV4_GTWY=""

nmcli connection modify "${NETWORK_CONNECTION}" ipv4.addresses "${MY_IPV4_ADDR}"
nmcli connection modify "${NETWORK_CONNECTION}" ipv4.dns "${IPV4_DNS}"
nmcli connection modify "${NETWORK_CONNECTION}" ipv4.gateway "${MY_IPV4_GTWY"
nmcli connection modify "${NETWORK_CONNECTION}" ipv4.ignore-auto-dns yes
nmcli connection modify "${NETWORK_CONNECTION}" ipv4.method manual


## OPTIONAL
IPV6_DNS="2606:4700:4700:0:0:0:0:1112,2606:4700:4700:0:0:0:0:1002,2001:4860:4860:0:0:0:0:8888,2001:4860:4860:0:0:0:0:8844"
MY_IPV6_ADDR=""
MY_IPV6_GTWY=""

nmcli connection modify "${NETWORK_CONNECTION}" ipv6.addresses "${IPV6_DNS}"
nmcli connection modify "${NETWORK_CONNECTION}" ipv6.dns "${MY_IPV6_ADDR}"
nmcli connection modify "${NETWORK_CONNECTION}" ipv6.gateway "${MY_IPV6_GTWY}"
nmcli connection modify "${NETWORK_CONNECTION}" ipv6.ignore-auto-dns yes
nmcli connection modify "${NETWORK_CONNECTION}" ipv6.method manual
```


### Reboot

```bash
systemctl reboot
```

---



