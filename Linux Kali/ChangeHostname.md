
## 🦄 How to Change Hostname in Kali Linux 

#### 1. Check Hostname

```bash
hostnamectl
```
#### 2. Set Hostname i.e devshomes

```bash
sudo hostnamectl set-hostname devhomes
```
#### 3. Update hosts File path: **/etc/hosts** file

```bash
sudo nano /etc/hosts
```

#### 4. Paste below code or write in same format
````bash
127.0.0.1       localhost
127.0.1.1       devhomes.local devhomes
# 192.168.29.10   devhomes.local devhomes
````
> [!NOTE] or if you want a server name like *dns*

````bash
127.0.0.1       localhost
127.0.1.1       dns.devhome.local dns
# 192.168.29.10   dns.devhome.local dns
````

#### 5. Check Hostname
```bash
cat /etc/hostname
```

> OR

```bash
echo "dns" | sudo tee /etc/hostname
```

#### 6. Restart hostname service *[ if needed ]*
```bash
sudo systemctl restart systemd-hostnamed
```

#### 7. Reboot System *[ if needed ]*
```bash
sudo reboot
```
#### 8. Verify Again

```bash
hostname
hostname -f
hostnamectl
```

> [!NOTE]
Recommended Hostname for DNS server setup : dns
FQDN: dns.devhome.local
IP: 192.168.29.10