
### 🐳 Docker: Portainer Installation Commands Listed Here

```bash
docker --version
```
```bash
sudo systemctl status docker
```
```powershell
sudo docker ps
```
## Install Portainer Using Docker Compose

>Create directory for portainer data:
```bash
sudo mkdir -p /opt/dockers/portainer
```
>goto directory:
```bash
cd /opt/dockers/portainer
```
>compose portainer yaml file for docker:
```bash
sudo nano docker-compose.yml
```
### Save this as `docker-compose.yml`:

```yml
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:lts
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - "9443:9443"
      - "8000:8000"  # Remove if you do not intend to use Edge Agents

volumes:
  portainer_data:
    name: portainer_data

networks:
  default:
    name: portainer_network
```
```bash
cat docker-compose.yml
```

>Then run to install Portainer:
```bash
sudo docker compose up -d
```
>[!NOTE]
>Your data is kept in the portainer_data volume.

### Open Browser:

```bash
https://localhost:9443
```
>OR `https://<SERVER-IP>:9443`</br>
>Admin account 5 minute ke andar bana lein, warna `docker restart portainer` karna padega.

## For Token: During Admin Account Creation:
Run this command:

```bash
docker logs portainer
```
> or
```bash
docker logs portainer 2>&1 | grep -i token
```

>[!IMPORTANT]
>You can find `setup_token=`. just copy Token ID and paste on portal's token field


### If token is expire or didn't visible:
Run these cmds:
```bash 
docker restart portainer
docker logs portainer
```
>or if run via `docker compose`

```bash
docker compose restart portainer
docker compose logs portainer
```

>[!NOTE]
>Token har restart par badal sakta hai, isliye hamesha latest logs se hi token lein. Admin account ban jaane ke baad token ki zarurat nahi padti.


## Troubleshooting
`Permission denied on docker commands: prefix with sudo or add your user to the docker group.
Can’t reach the UI: check that port 9443 isn’t blocked by a firewall (e.g. sudo ufw allow 9443).
Security: mounting docker.sock gives Portainer full control of Docker, so don’t expose the UI publicly without strong credentials, and ideally put it behind a VPN or reverse proxy.`
 
 ## Docker Useful Commands:
|Use For    |Command|
|Start	    |sudo docker compose up -d|
|Stop	    |sudo docker compose stop|
|Restart	|sudo docker compose restart|
|Logs	    |sudo docker compose logs -f|
|Status	    |sudo docker compose ps|