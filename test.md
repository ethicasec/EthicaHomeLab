# My Project

> A short description of what this project does.

## 📋 Overview

This project provides a simple solution for managing DNS
and network configuration.

## ✨ Features

- DNS filtering
- Local DNS resolution
- Network monitoring
- HTTPS support
- Easy configuration

## 🛠️ Requirements

- Windows Server 2022
- VMware Workstation
- 4 GB RAM
- 20 GB storage
- Static IP address

## 📦 Installation

### 1. Download the software

Download the required software from the official website.

### 2. Install the application

Run the installer:

```powershell
setup.exe
```

### 3. Configure the server

Open the configuration file:

```text
C:\ProgramData\Example\config.yaml
```

## ⚙️ Configuration

Example configuration:

```yaml
server:
  address: 192.168.29.10
  port: 53

dns:
  enabled: true
  filtering: true
```

> [!NOTE]
> Make sure the server has a static IP address.

## 🧪 Testing

Run:

```bash
nslookup example.com
```

Expected result:

```text
Server:  192.168.29.10
Address: 192.168.29.10
```

## 📊 Configuration Table

| Setting | Value | Description |
|---------|-------|-------------|
| IP | 192.168.29.10 | DNS Server |
| Port | 53 | DNS Port |
| Protocol | DNS | Name resolution |

## 🔧 Troubleshooting

<details>
<summary>DNS is not working</summary>

Check whether the DNS service is running:

```powershell
Get-Service DNS
```

Also verify that port `53` is not blocked.

</details>

## 🚀 Future Improvements

- [ ] HTTPS support
- [ ] DoH support
- [ ] DoT support
- [ ] Monitoring dashboard

## 📄 License

This project is licensed under the MIT License.

## 👤 Author

Your Name

---

⭐ If you find this project useful, consider giving it a star!