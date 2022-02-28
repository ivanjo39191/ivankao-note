# Ubuntu docker-compose問題

Created: February 1, 2022 5:22 PM
Tags: OS

```jsx
sudo snap remove docker
sudo apt install docker.io docker-compose

sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo rm /usr/bin/docker-compose
```

```jsx
# use
/usr/bin/docker
/usr/local/bin/docker-compose

```

[https://github.com/docker/compose/issues/6361](https://github.com/docker/compose/issues/6361)