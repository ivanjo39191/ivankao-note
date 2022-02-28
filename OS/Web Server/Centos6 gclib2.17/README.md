# Centos6 gclib2.17

Created: February 1, 2022 5:22 PM
Tags: OS

因為 Python 套件 pillow8.4.0 需求 gclib2.17，在安裝上遇到一些問題

以下是最後安裝 rpm 進行升級的執行程式

[https://gist.github.com/harv/f86690fcad94f655906ee9e37c85b174](https://gist.github.com/harv/f86690fcad94f655906ee9e37c85b174)

**[glibc-2.17_centos6.sh](https://gist.github.com/harv/f86690fcad94f655906ee9e37c85b174#file-glibc-2-17_centos6-sh)**

```python
#! /bin/sh

# update glibc to 2.17 for CentOS 6

wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-2.17-55.el6.x86_64.rpm
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-common-2.17-55.el6.x86_64.rpm
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-devel-2.17-55.el6.x86_64.rpm
wget http://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-headers-2.17-55.el6.x86_64.rpm
wget https://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-utils-2.17-55.el6.x86_64.rpm
wget https://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/glibc-static-2.17-55.el6.x86_64.rpm
sudo rpm -Uvh glibc-static-2.17-55.el6.x86_64.rpm \
glibc-utils-2.17-55.el6.x86_64.rpm --force --nodeps
sudo rpm -Uvh glibc-2.17-55.el6.x86_64.rpm --force --nodeps \
glibc-common-2.17-55.el6.x86_64.rpm --force --nodeps \
glibc-devel-2.17-55.el6.x86_64.rpm --force --nodeps \
glibc-headers-2.17-55.el6.x86_64.rpm --force --nodeps
```