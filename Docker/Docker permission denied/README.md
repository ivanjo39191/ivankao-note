# Docker permission denied

Created: February 1, 2022 5:22 PM
Tags: Docker

```python
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 666 /var/run/docker.sock
```