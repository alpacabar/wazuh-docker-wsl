# Wazuh Single-Node Setup (Docker + WSL)

A clean and tested guide to deploy Wazuh in a single-node environment with Docker, inside WSL.

## 0. Prerequisites
### Enable WSL (Windows Subsystem for Linux)
- Open PowerShell as Administrator:
```
wsl --install
```
- Reboot when prompted
### Install Ubuntu 
### Install Docker Desktop (for Windows)
- Download and install from
- Enable WSL Integration in Docker settings:
    - Settings -> Resources -> WSL Integration -> Enable for your Ubuntu distro
### Set Up Docker in WSL 
```
sudo usermod -aG docker $USER
newgrp docker
```
- Test Docker inside WSL
```
docker run hello-world
```
## 1. Clone the Official Wazuh Docker Repository
```
cd ~
git clone https://github.com/wazuh/wazuh-docker.git
cd wazuh-docker/single-node
```
## 2. Generate TLS Certificates (Using Wazuh Indexer)
### Create a generate-certs.yml file 
```
version: '3.9'
services:
  generator:
    image: wazuh/wazuh-indexer:4.7.2
    command: >
      bash -c "
      mkdir -p /certs &&
      cp -r /usr/share/wazuh-indexer/certs/* /certs/"
    volumes:
      - ./config/certs:/certs
```
### Run the generator
```
sudo rm -rf config/certs
mkdir -p config/certs
sudo chmod 777 config/certs

docker-compose -f generate-certs.yml run --rm generator
```
## 3. Pull Required Docker Images
```
docker-compose pull
```
## 4. Launch Wazuh Stack
```
docker-compose up -d
```
- Make sure the following containers are running:
    - wazuh.indexer
    - wazuh.manager
    - wazuh.dashboard
 
  Check with:
```
docker ps
```
## 5. Initialize OpenSearch Security Plugin
### Run Securitty Admin Tool:
```
docker exec -it single-node_wazuh.indexer_1 bash
export JAVA_HOME=/usr/share/wazuh-indexer/jdk
chmod +x /usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh

/usr/share/wazuh-indexer/plugins/opensearch-security/tools/securityadmin.sh \
  -cd /usr/share/wazuh-indexer/plugins/opensearch-security/securityconfig \
  -icl -nhnv \
  -cacert /usr/share/wazuh-indexer/certs/root-ca.pem \
  -cert /usr/share/wazuh-indexer/certs/admin.pem \
  -key /usr/share/wazuh-indexer/certs/admin-key.pem
```
You should see success messages for uploading all security config files
Exit the container:
```
exit
```
## 6. Verify OpenSearch Status
```
curl -k -u admin:admin https://localhost:9200/_cluster/health?pretty
```
Expected output: "status" : "green"

## 7. Access the Wazuh Dashboard
Visit in your browser:
```
https://localhost
```
Login:
```
Username: admin
Password: admin
```

