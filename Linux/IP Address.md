
### 🐳 Configure IP Address Gateway DNS in ubuntu

Step 1: Find your network interface name

```bash
ip a
```
Step 2: List your netplan config files

```bash
ls /etc/netplan/
```

ethca@ubuntu:/etc/netplan$ sudo cat 90-NM-ea056b1c-fd33-4d1d-ab16-3b40bd3a3e01.yaml 
network:
  version: 2
  ethernets:
    NM-ea056b1c-fd33-4d1d-ab16-3b40bd3a3e01:
      renderer: NetworkManager
      match:
        name: "ens33"
      addresses:
      - "192.168.29.40/24"
      gateway4: 192.168.29.1
      dhcp6: true
      wakeonlan: true
      networkmanager:
        uuid: "ea056b1c-fd33-4d1d-ab16-3b40bd3a3e01"
        name: "Profile 1"
        passthrough:
          ethernet._: ""
          ipv6.addr-gen-mode: "default"
          ipv6.ip6-privacy: "-1"
          proxy._: ""
