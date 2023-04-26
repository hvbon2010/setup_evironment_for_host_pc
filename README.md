# Introduce
Setup for basic tools with new host PC: ssh, vim, tmux, ...

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
 dns-domain 192.168.0.1 8.8.8.8
 dns-nameservers 192.168.0.1 4.4.4.4
```

Restart network service

`sudo systemctl restart networking.service`

If this change do not effect, so, we need to reboot OS

`sudo reboot`

# Check WAN connection
`ping -c 1 www.google.com`

## Good connection:

![image](https://user-images.githubusercontent.com/32226325/208014273-49c1d868-7ebe-409b-b27d-48be6c5028c0.png)

## Can not resolved DNS:

![image](https://user-images.githubusercontent.com/32226325/208014343-8b70d75c-91d9-4459-8de9-f4c79bcf845f.png)

How to fix:

Add google dns `8.8.8.8` and `4.4.4.4` to resolv config file

`sudo mkdir -p /etc/systemd/resolved.conf.d`

`sudo nano /etc/systemd/resolved.conf.d/dns_servers.conf`

```
[Resolve]
DNS=8.8.8.8 4.4.4.4
Domains=~.
```

`sudo reboot`

# Remote with SSH key
On a client PC, you need to generate a SSH key pair

`ssh-keygen -t ed25519`

Copy private key to host PC

`ssh-copy-id -i ~/.ssh/id_ed25519_[xxx] [host_name]@[ip_address]`

Remote

`ssh [host_name]@[ip_address] -i ~/.ssh/id_ed25519_[xxx].pub`

# Secure host PC with SSH authentication
Change SSH config

`sudo vim /etc/ssh/sshd_config`

- Change SSH port from 22 to other

<img width="319" alt="image" src="https://user-images.githubusercontent.com/32226325/193379357-b2a82e77-2055-4451-9d80-172ee7b8bdad.png">

- Change authentication options (Decrease authengracetime, number of ssh sessions, authen tries. Use authorize key) 

<img width="541" alt="image" src="https://user-images.githubusercontent.com/32226325/193379344-0e6d9745-63e3-4f46-9064-1d22a468ebc7.png">

- Disable authen with password

<img width="478" alt="image" src="https://user-images.githubusercontent.com/32226325/193379373-c92cf6c2-5c18-45a8-9dad-4ef06574e1ea.png">

Restart SSH service

`sudo systemctl restart ssh.service`

If this change do not effect, so, we need to reboot OS

`sudo reboot`

# Change date time
`timedatectl set-timezone Asia/Ho_Chi_Minh`

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

Verify this script is worked:

```
sudo ifconfig eth0 down
sudo ifconfig eth1 down
sudo ifconfig eth0 up
sudo ifconfig eth1 up
```

`ip route show`

<img width="553" alt="image" src="https://user-images.githubusercontent.com/32226325/183931406-76a8c480-1ce0-47d1-a0f8-41ffdd72a215.png">

# Auto ssh a host with 2 addresses ip
Linux:

```
# any amount of these is possible
Match originalhost FW_2 exec "nc -v -z -w 1 10.148.4.33 22 2>/dev/null"
    User hvbon
    HostName 10.148.4.33

# fallback
Host FW_2
    User hvbon
    HostName 192.168.50.2
```

MACOS: change nc option `-w` to `-G`

```
# any amount of these is possible
Match originalhost FW_2 exec "nc -v -z -G 1 10.148.4.33 22 2>/dev/null"
    User hvbon
    HostName 10.148.4.33

# fallback
Host FW_2
    User hvbon
    HostName 192.168.50.2
```
