## How to setup Parsec, Moonlight using Tailscale
## And use a VPS as relay server to reduce latency
---
This guide will help you to:
- Resolve 6023 error on Parsec.
- Setup port forwarding in a VPS.
- Setup Tailscale to work with Parsec and Moonlight.
- Open port without modify router's config.

When do you need/follow this guide:
- Setup your PC as host for low latency game streaming.
- You can't setup Port Forwarding.
- You're not the owner of the network.
- You don't have the permission to config the network.
- You can PAY about 1-2$ per month for renting a VPS.

---

## Installation

#### Install Tailscale, Moonlight and/or Parsec

- [Install Tailscale](https://tailscale.com/kb/1347/installation)
- [Install Moonlight](https://github.com/moonlight-stream/moonlight-docs/wiki/Setup-Guide)
- [Install Parsec](https://parsec.app/downloads)</br>
Install this on both your host and client pc.

#### Now, you need to change to config file of Parsec on the host PC:
1. Exit Parsec, this is required before change the json file.
2. Open the json file:
- Press Win+R
- Insert: ``` %AppData% ```
- Open "Parsec" folder
- Add this to file "config.json" 

```
"app_custom_address": {
    "value": "<ip-address>"
},
```

The "value" is the Tailscale ip address of the host PC. Example:

```
"app_custom_address": {
    "value": "100.64.0.3"
},
```

3. Save the file and start Parsec.

- NOTE that if you want to go back to normal Parsec connection without using Tailscale, you need to remve this config.

- From here, you can connect to the host PC with Parsec without 6023 error.

#### For Moonlight

You just need to add new PC using the Tailscale ip address of the VPS.

---

After done above steps, now you can connect to the host without any error. But you may face the high latency problem. Now, it's time to setup the VPS as relay server to reduce the latency.

---


#### Setup VPS

- You must rent a VPS (or some provider can give you a free trial).
- Choose the provider that place their server on your country/ city because this will give you the lowest ping.
- Choose the lowest configuration because this don't need a high performance and this will save your money. In my case, I choose 1 CPU core, 1 GB of RAM, 100 Mbps with unlimited data usage.
- Choose the Linux as Operating System. Ubuntu is recommended.

---

Test the ping of the VPS. In my case:
```
Host PC <-> Client PC = 100ms :(
Host PC <-> VPS       = 4ms
VPS     <-> Client PC = 4ms
```

Now go to the VPS console:

- Update your VPS:

```
sudo apt update
sudo apt upgrade
```

- [Install Tailscale for Linux](https://tailscale.com/kb/1031/install-linux)


- You may got this warning, do as follow before starting tailscale:

Warning: IP forwarding is disabled, subnet routing/exit nodes will not work.

```
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

Warning: UDP GRO forwarding is suboptimally configured on eth0, UDP forwarding throughput capability will increase with a configuration change.

```
NETDEV=$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")
sudo ethtool -K $NETDEV rx-udp-gro-forwarding on rx-gro-list off
```
```
printf '#!/bin/sh\n\nethtool -K %s rx-udp-gro-forwarding on rx-gro-list off \n' "$(ip -o route get 8.8.8.8 | cut -f 5 -d " ")" | sudo tee /etc/networkd-dispatcher/routable.d/50-tailscale
sudo chmod 755 /etc/networkd-dispatcher/routable.d/50-tailscale
```
```
sudo /etc/networkd-dispatcher/routable.d/50-tailscale
test $? -eq 0 || echo 'An error occurred.'
```

- Start Tailscale:
```
sudo tailscale up
```

- Now check the connection status:
```
sudo tailscale status
```

This is my result (I hide some information):
```
100.64.0.6      ubuntu              linux   idle; offers exit node
100.64.0.3      host-pc             windows -
100.64.0.4      phone               android 
```

```
100.64.0.6  // This is my VPS address
100.64.0.3  // Host PC address
100.64.0.4  // Client PC address
```

- Now check the connection status again:



- This mean my phone (as Client PC) traffic now go through the VPS.

---
#### Setup port forwarding in VPS

Install netfilter-persistent to save your config if VPS shutdown/ reboot:
```
sudo apt install netfilter-persistent -y
```

Install iptables-persistent
```
sudo apt install iptables-persistent
sudo iptables-save > /etc/iptables/rules.v4
```


Enable port forward
```
sudo nano /etc/sysctl.conf
```

Remove # in this line
```
net.ipv4.ip_forward=1
```
Press Ctrl+O -> Enter to save then press Ctrl+X

Apply
```
sudo sysctl -p
```

- Now forward a port to Host PC, in my case, I use port 9000:
```
sudo iptables -t nat -A PREROUTING -p udp --dport 9000 -j DNAT --to-destination 100.64.0.3:9000
```

In here, "udp" is the protocol. Some case it will be "tcp"</br>
"9000" is the port number</br>
"100.64.0.3" is the Host PC ip address used in Tailscale


- Save and reload the config
```
sudo netfilter-persistent save && sudo netfilter-persistent reload
```
- Check if rule exist:
```
sudo iptables-save
```

---
Now change the port number in Parsec:
- Host PC, change the "Host Start Port" to 9000
- Client PC, change "Client Port" to 9000

Start connect and check the ping.

---
#### For Moonlight, you need to forward the following port:
TCP 47984, 47989, 48010</br>
UDP 47998, 47999, 48000, 48002, 48010</br>

- Simply use following command:
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 47984 -j DNAT --to-destination 100.64.0.3:47984
sudo iptables -t nat -A PREROUTING -p tcp --dport 47989 -j DNAT --to-destination 100.64.0.3:47989
sudo iptables -t nat -A PREROUTING -p tcp --dport 48010 -j DNAT --to-destination 100.64.0.3:48010

sudo iptables -t nat -A PREROUTING -p udp --dport 47998 -j DNAT --to-destination 100.64.0.3:47998
sudo iptables -t nat -A PREROUTING -p udp --dport 47999 -j DNAT --to-destination 100.64.0.3:47999
sudo iptables -t nat -A PREROUTING -p udp --dport 48000 -j DNAT --to-destination 100.64.0.3:48000
sudo iptables -t nat -A PREROUTING -p udp --dport 48002 -j DNAT --to-destination 100.64.0.3:48002
sudo iptables -t nat -A PREROUTING -p udp --dport 48010 -j DNAT --to-destination 100.64.0.3:48010
```
---
~Good luck!
---
