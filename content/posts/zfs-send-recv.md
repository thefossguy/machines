---

title: "ZFS send and recv"
date: 2022-07-23T07:00:00+05:30
draft: false
toc: true

---

## Step 0000: Configure SSH Access

**On the sender's system:**

```bash
cd ~/.ssh/
ssh-keygen -t ed25519 -f zfs_send
```

```bash
echo "
Host hostname.localdomain
        Hostname <ip-addr>
        User root
        IdentityFile ~/.ssh/zfs_send
        Port <port>
" | tee -a ~/.ssh/config
```

---

**On the receiving system:**

`sudoedit /etc/ssh/sshd_config`

```
[...]
PermitRootLogin yes
[...]
```

```bash
user $ sudo -i

root # echo "zfs_send.pub" | tee -a ~/.ssh/authorized_keys
```

`sudo systemctl restart sshd`

**This guide was possible because of the _[amazing documentation of Klara Systems](https://klarasystems.com/articles/introduction-to-zfs-replication/)_.**


## Step 0001: Delegate ZFS administration permissions to unprivileged users

**On the sender's system:**

```bash
sudo zfs allow -u USER send,snapshot,hold POOL
```

**On the receiving end:**

```bash
sudo zfs allow -u USER compression,mountpoint,create,mount,receive POOL
```

## Step 0010: Send

```zfs
sudo zfs snapshot <vol>@sendtest
zfs send -v <vol>@sendtest | ssh hostname.localdomain zfs receive <pool>/backups/<vol-name>
```
