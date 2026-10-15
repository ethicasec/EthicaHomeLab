
### 🐳 Docker: Portainer Installation Commands Listed Here

#### 1. Stop Portainer:

```bash
sudo docker stop <portainer_name_or_id>
```

#### 2. Reset Portainer Password:
```bash
sudo docker run --rm -v portainer_data:/data portainer/helper-reset-password
```
#### 3. Result Show on Screen like this
    Status: Downloaded newer image for portainer/helper-reset-password:latest
    {"level":"info","filename":"portainer.db","time":"2026-10-04T16:39:49Z","message":"loading PortainerDB"}
    2026/10/04 16:39:50 Password successfully updated for user: admin
    2026/10/04 16:39:50 Use the following password to login: TPF7R]/m?.03upofNlA=,s5VU1_6-29S
    {"level":"info","time":"2026-10-04T16:39:50Z","message":"closing PortainerDB"}

>[!IMPORTANT]
>Kindly note Temporary Password and use with your user account and also change it.

#### 4. Start Portainer:
```bash
docker start <portainer_name_or_id>
```
