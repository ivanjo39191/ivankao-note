# Install Docker Compose with Linux

Created: February 1, 2022 5:22 PM
Tags: Docker

Install 

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

Upgrading

```bash
docker-compose migrate-to-labels
docker container rm -f -v myapp_web_1 myapp_db_1 ...

```

Uninstallation

```bash
sudo rm /usr/local/bin/docker-compose
```

[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)