# VPN_Wireguard_Hosting

## Welcome to my tutorial about: How to install and configure Wireguard VPN securely (encryption is out of the box from Wireguard)

### Installing WireGuard (Ubuntu 24.04 LTS)

```
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install wireguard
```

### Firewall
```
sudo ufw allow ssh
sudo ufw allow 51820/udp
sudo ufw enable
sudo ufw status
```

### IP forwarding

```
sudo vim /etc/sysctl.conf
```

uncomment net.ipv4.ip_forward=1

```
sudo sysctl -p
```

output should be `net.ipv4.ip_forward=1`

### Generate Keys

On server:
```
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
sudo cat /etc/wireguard/publickey
sudo cat /etc/wireguard/privatekey
```

On client:
```
mkdir -p ~/wireguard/clients/client1
cd ~/wireguard/clients/client1
chmod 700 ~/wireguard/clients/client1
chmod 600 ~/wireguard/clients/client1/client_privatekey
wg genkey | tee client_privatekey | wg pubkey > client_publickey
sudo cat client_privatekey
sudo cat client_publickey
```

### On VPN Server: Add connection config file

check your network interface:
```
ip route list default
```

```
vim /etc/wireguard/wg0.conf
```

add the following content and replace `eth0` with your network interface and `<server-privatekey>`,`<client-publickey>` with your keys:
```
[Interface]
PrivateKey = <server-privatekey>
Address = 10.0.0.1/24
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820
```

### NEXT: add client to your config(if adding more clients chage ip address e.g. 10.0.0.2/32... 10.0.0.3/32... 10.0.0.4/32):
```
vim /etc/wireguard/wg0.conf
```
```
[Peer]
PublicKey = <client-publickey>
AllowedIPs = 10.0.0.2/32
```

### On client: add connection config file

```
vim /etc/wireguard/wg0.conf
```

```
[Interface]
Address = 10.0.0.2/32
PrivateKey = <client-privatekey>
DNS = 1.1.1.1

[Peer]
PublicKey = <server-publickey>
Endpoint = <server-public-ip>:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

On VPN server do:

```
wg-quick up wg0
```

output example:
```
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.0.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

check configuration:
```
wg show
```

output example:
```
interface: wg0
  public key: pcDxSxSZp5x87cNoRJaHdAOzxrxDfDUn7pGmrY/AmzI=
  private key: (hidden)
  listening port: 51820

peer: gCQKfJL8Xff2MNmvceVQ0nQAmLsSM0tXClhvVNzSil4=
  allowed ips: 10.0.0.2/32
```

enable on VPN on boot
```
systemctl enable wg-quick@wg0
```

### On Client Server: Start vpn

```
sudo wg-quick up wg0                # or sudo wg-quick up ./wg0.conf
#systemctl enable wg-quick@wg0      # to enable start on boot
```


stop vpn connaction and disable boot
```
sudo wg-quick down wg0
sudo systemctl stop wg-quick@wg0
```
