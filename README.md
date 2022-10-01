# Introduce
Setup for basic tools with new host PC: ssh, vim, tmux, ...

# Setup ssh

# Setup vim
Extract .vim.tar.gz to home folder.

# Setup tmux
`sudo apt-get install tmux`

`git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm`

`Copy .tmux.conf to home folder.`

# Setup a fixed IP address
Get network interface you want to fix IP address first

`ifconfig`

`sudo vim /etc/network/interfaces`

Add below code to end of this file

```
# Fixed an IP address for eth0
auto eth0
iface eth0 inet static
 address 192.168.0.102
 netmask 255.255.255.0
 gateway 192.168.0.1
 dns-domain 192.168.0.1
 dns-nameservers 192.168.0.1
```

Restart network service

`sudo systemctl restart networking.service`

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

Verify this script is worked:

```
sudo ifconfig eth0 down
sudo ifconfig eth1 down
sudo ifconfig eth0 up
sudo ifconfig eth1 up
```

`ip route show`

<img width="553" alt="image" src="https://user-images.githubusercontent.com/32226325/183931406-76a8c480-1ce0-47d1-a0f8-41ffdd72a215.png">


