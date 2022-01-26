# nanopi-r1

NanoPi R1 Armbian commands

Purpose of this README.md is to configure NanoPi-R1 with a WAN, LAN and WLAN (AP)

## Initial setup

```txt
ssh root@172.31.85.189

password: 1234
```

It will ask to change `root` password and create a new user.

Write the system to `eMMC` storage.

```txt
nand-sata-install
```

and select

```txt
Boot from eMMC - system on eMMC
```

Edit `sudo` to allow you run commands without password.

```txt
visudo
```

and change the line

```txt
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

I don't like another editor but vim, so let's change it.

```txt
sudo update-alternatives --config editor
```

And select `vim.basic`

```txt
There are 3 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
  0            /bin/nano            40        auto mode
  1            /bin/nano            40        manual mode
  2            /usr/bin/mcedit      25        manual mode
* 3            /usr/bin/vim.basic   30        manual mode
```

Also, by default, when you use mouse to select from `vim`, it will enter in the visual mode, which is bad in my POV. So, let's disable `vim` visual mode.

```txt
echo "set mouse-=a" >> ~/.vimrc
```

Do not forget about the `root` user.

```txt
sudo su - -c 'echo "set mouse-=a" > /root/.vimrc'
```

## Keep things update

It is always nice to keep the thing update.

```txt
sudo apt update
sudo apt full-upgrade
```

## Create a shared connection

Install dnsmasq and enable it.

```txt
sudo apt install dnsmasq
```

Create the file `/etc/dnsmasq.d/default.conf`

```txt
sudo vi /etc/dnsmasq.d/default.conf
```

And add the content below:

```conf
# Set the interface on which dnsmasq operates.
# If not set, all the interfaces is used.
interface=eth1,wlan0

# To disable dnsmasq's DNS server functionality.
port=0

# To enable dnsmasq's DHCP server functionality.
dhcp-range=subnet1,172.31.86.220,172.31.86.250,255.255.255.0,12h
dhcp-range=subnet2,172.31.87.220,172.31.87.250,255.255.255.0,12h

# Set static IPs of other PCs
#dhcp-host=90:9f:44:d8:16:fc,iptime,172.31.86.220,infinite

# Set gateway as Router.
dhcp-option=subnet1,3,172.31.86.1
dhcp-option=subnet2,3,172.31.87.1

# Set DNS server as Router.
dhcp-option=subnet1,6,8.8.8.8,8.8.4.4
dhcp-option=subnet2,6,8.8.8.8,8.8.4.4

# Logging.
log-facility=/var/log/dnsmasq.log   # logfile path.
log-async
log-queries # log queries.
log-dhcp    # log dhcp related messages.
```

Configure interfaces.

```txt
nmcli c add type wifi ifname wlan0 con-name wlan autoconnect yes ssid nanopi_r1 mode ap
nmcli c modify wlan 802-11-wireless.band bg wifi-sec.key-mgmt wpa-psk wifi-sec.psk "YourPasswordHere" ipv4.method shared ipv4.addresses 172.31.87.1/24
nmcli c up wlan
nmcli c modify "Wired connection 1" con-name wan
nmcli c modify "Wired connection 2" ifname eth1 ipv4.method shared con-name lan ipv4.addresses 172.31.86.1/24
nmcli c up lan
```

Enable and (re)start `dnsmasq`

```txt
sudo systemctl enable dnsmasq
sudo systemctl restart dnsmasq
```

## OpenVPN Client

Install OpenVPN

```txt
sudo apt install openvpn
```

Copy your VPN config file to `/etc/openvpn/client`

It is important to be a `.conf` file. The name `protonvpn` is a name of my vpn, you can change to whatever you want. Just keep in mind that you need to change to enable and start the service.

```txt
vi /etc/openvpn/client/protonvpn.conf
```

To enable and start it:

```txt
sudo systemctl enable openvpn-client@protonvpn
sudo systemctl start openvpn-client@protonvpn
```
