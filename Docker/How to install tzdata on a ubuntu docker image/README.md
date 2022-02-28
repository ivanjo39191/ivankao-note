# How to install tzdata on a ubuntu docker image

Created: February 1, 2022 5:22 PM
Tags: Docker

Dockerfile 安裝 ubuntu 套件時停在選擇時區

```python
# Dockfile
RUN apt-get install -y tzdata

#####################################
Please select the geographic area in which you live. Subsequent configuration
questions will narrow this down by presenting a list of cities, representing
the time zones in which they are located.

  1. Africa      4. Australia  7. Atlantic  10. Pacific  13. Etc
  2. America     5. Arctic     8. Europe    11. SystemV
  3. Antarctica  6. Asia       9. Indian    12. US
Geographic area:
``
```

解法:

```python
# Dockerfile
ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"
```

[https://serverfault.com/questions/949991/how-to-install-tzdata-on-a-ubuntu-docker-image](https://serverfault.com/questions/949991/how-to-install-tzdata-on-a-ubuntu-docker-image)