## Install Docker
### Uninstall conflicting packages
```bash
for  pkg  in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
sudo  apt-get  remove  -y  $pkg
done
```
 ## Install Docker dependencies
```bash
sudo  apt-get  update
sudo  apt-get  install  -y  ca-certificates  curl
```
## Add Docker's official GPG key
```bash
sudo  install  -m  0755  -d  /etc/apt/keyrings
sudo  curl  -fsSL  https://download.docker.com/linux/ubuntu/gpg  -o  /etc/apt/keyrings/docker.asc
sudo  chmod  a+r  /etc/apt/keyrings/docker.asc
```
## Add the Docker repository to Apt sources
```bash
echo  \
"deb [arch=$(dpkg  --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo  "$VERSION_CODENAME") stable"  |  \
sudo  tee  /etc/apt/sources.list.d/docker.list  >  /dev/null
sudo  apt-get  update
```
# Install Docker packages
 ```bash
sudo  apt-get  install  -y  docker-ce  docker-ce-cli  containerd.io  docker-buildx-plugin  docker-compose-plugin
```
# Install Docker-compose
```bash
sudo  apt-get  update
sudo  apt  install  docker-compose
```

# Add current user to the docker group
```bash
sudo  groupadd  docker
sudo  usermod  -aG  docker  $USER