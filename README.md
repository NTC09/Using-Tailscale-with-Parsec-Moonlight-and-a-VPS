## How to setup a VPS as relay server for using Tailscale with Parsec, Moonlight
---
This guide will help you to:
- Resolve 6023 error on Parsec.
- Setup port forwarding in a VPS.
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
- [Install Parsec](https://parsec.app/downloads)

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

You just need to add new PC using the Tailscale ip address.

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
*Host PC <-> Client PC = 100ms :(
*Host PC <-> VPS       = 4ms
*VPS     <-> Client PC = 4ms
```

Now go to the VPS console:

- Update your VPS:

```
sudo apt upgrade
sudo apt upgrade
```

- [Firstly, install Tailscale for Linux](https://tailscale.com/kb/1031/install-linux)
- 
