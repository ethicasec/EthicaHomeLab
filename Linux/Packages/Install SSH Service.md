
### 🐳 Install and Enable SSH Services in ubuntu

#### Step 1: Find SSH service install or not

```bash
sudo systemctl status ssh
```
> #### or 

```bash
dpkg -l | grep openssh-server
```

#### Step 2: Install Open SSH Server

```bash
sudo apt update
sudo apt install openssh-server -y
```
#### Step 3: Check and Enable SSH Service on Startup

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```
#### Step 3: Allow SSH on Firewall

```bash
sudo ufw allow ssh
sudo ufw status
```
