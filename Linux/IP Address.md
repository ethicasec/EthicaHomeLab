
### 🐳 Configure IP Address Gateway DNS in ubuntu

Step 1: Find your network interface name

```bash
ip -br addr
```
#### Step 2: Goto netplan directory List your netplan config files

```bash
cd /etc/netplan/ && ls
```

#### Step 3: find a file like name 00-....yaml

```bash
sudo nano 00-installer-config.yaml 
```
>[!NOTE]
> Change ipv4 and ipv6 status true to false save and restart Network Manager

>[!IMPORTANT]
> For Set static  ip check below code

ubuntu:/etc/netplan$ sudo cat 90-NM-ea056b1c-fd33-4d1d-ab16-.yaml 

```bash
network:
  version: 2
  ethernets:
    NM-ea056b1c-fd33-4d1d-ab16-3b40bd:
      renderer: NetworkManager
      match:
        name: "ens33"
      addresses:
      - "192.168.29.15/24"
      gateway4: 192.168.29.1
      dhcp6: true
      wakeonlan: true
      networkmanager:
        uuid: "ea056b1c-fd33-4d1d-ab16-"
        name: "Profile 1"
        passthrough:
          ethernet._: ""
          ipv6.addr-gen-mode: "default"
          ipv6.ip6-privacy: "-1"
          proxy._: ""
```