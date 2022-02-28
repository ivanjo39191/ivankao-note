# sqlite 升級

Created: February 1, 2022 5:22 PM
Tags: Django, OS

[https://zhuanlan.zhihu.com/p/110743704](https://zhuanlan.zhihu.com/p/110743704)

```bash
# 一.wget升级
yum install -y wget
# 二.sqlite3安装
sudo yum install sqlite-devel
sqlite3 -version
# 三.sqlite3升级
wget https://www.sqlite.org/2019/sqlite-autoconf-3290000.tar.gz
tar zxvf sqlite-autoconf-3290000.tar.gz
cd sqlite-autoconf-3290000/
./configure --prefix=/usr/local
make && make install

# 移除舊版本，替換為新版本
mv **/usr/bin/sqlite3 /usr/bin/sqlite3_old
ln -s /usr/local/bin/sqlite3 /usr/bin/sqlite3**
echo "/usr/local/lib" > /etc/ld.so.conf.d/sqlite3.conf
ldconfig
sqlite3 -version

```

No module named _sqlite3

[https://stackoverflow.com/questions/1210664/no-module-named-sqlite3](https://stackoverflow.com/questions/1210664/no-module-named-sqlite3)

```python
./configure --enable-loadable-sqlite-extensions && make && sudo make install
```