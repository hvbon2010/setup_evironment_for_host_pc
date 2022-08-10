# Introduce
Setup for basic tools with new host PC: ssh, vim, tmux, ...

# Setup ssh

# Setup vim
Extract .vim.tar.gz to home folder.

# Setup tmux
`sudo apt-get install tmux`

`git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm`

`Copy .tmux.conf to home folder.`

# Setup dual network for server
If we have a server with multiple network interface, 2 LANs or 1 LAN - 1 WIFI. And we want to use one network for local company network, and other for 
public network, we need to route the IPs to the suitable network.

Add this below script to `/etc/network/if-up.d`, this script will be auto call when any network interface up.

```
#!/bin/sh

PRIVATE=eth0
PUBLIC=eth1

if [ -x /bin/ip ]; then
  echo "Interface $IFACE up"
  if [ "$IFACE" = "$PRIVATE" ]; then
    /bin/ip route add 10.148.0.0/14 dev $PRIVATE
  elif [ "$IFACE" = "$PUBLIC" ]; then
    /bin/ip route add default dev $PUBLIC
  fi
fi
```

`/bin/ip route add 10.148.0.0/14 dev $PRIVATE`: Route all ip 10.148.x.x to private network.

`/bin/ip route add default dev $PUBLIC`: Route other IPs to public network.
