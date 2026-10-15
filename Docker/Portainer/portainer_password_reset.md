
### 🐳 Docker: Portainer Installation Commands Listed Here

```bash
sudo docker stop <portainer_name_or_id>
```
```bash
sudo docker run --rm -v portainer_data:/data portainer/helper-reset-password
```

>[!NOTE]
>Token har restart par badal sakta hai, isliye hamesha latest logs se hi token lein. Admin account ban jaane ke baad token ki zarurat nahi padti.


## Result Show on Screen like this
    Status: Downloaded newer image for portainer/helper-reset-password:latest
    {"level":"info","filename":"portainer.db","time":"2026-10-04T16:39:49Z","message":"loading PortainerDB"}
    2026/10/04 16:39:50 Password successfully updated for user: admin
    2026/10/04 16:39:50 Use the following password to login: TPF7R]/m?.03upofNlA=,s5VU1_6-29S
    {"level":"info","time":"2026-10-04T16:39:50Z","message":"closing PortainerDB"}

>[!IMPORTANT]
>Kindly note Temporary Password and use with your user account and also change it.

```bash
docker start <portainer_name_or_id>
```
