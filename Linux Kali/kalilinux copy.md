
### 🐳 Change Hostname in Kali Linux

#### CHECK HOSTNAME

```bash
hostnamectl
```
#### SET HOSTNAME i.e

```bash
sudo hostnamectl set-hostname devhomes
```
#### UPDATE /etc/hosts file
```bash
sudo nano /etc/hosts
```

#### Paste below code or write in same format
````bash
127.0.0.1       localhost
127.0.1.1       devhomes.local devhomes
# 192.168.29.76   devhomes.local devhomes
````
#### OR IF YOU WANT DNS HOSTNAME
````bash
127.0.0.1       localhost
127.0.1.1       dns.devhome.local dns
# 192.168.29.76   dns.devhome.local dns
````

#### UPDATE /etc/hosts file
```bash
cat /etc/hostname
```
OR
echo "dns" | sudo tee /etc/hostname

RESTART SERVICE 
```bash
sudo systemctl restart systemd-hostnamed
```

REBOOT SYSTEM
```bash
sudo reboot
```
VERIFY AGAIN

```poewrshell
hostname
hostname -f
hostnamectl
```