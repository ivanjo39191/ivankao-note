# Ubuntu 22.02 LTS 創建新使用者並開啟 ssh 登入

Created: June 28, 2022 10:23 AM  
Property: Ivan Kao  
Status: 已公開  
Tags: Linux  

![Untitled](images/Untitled.png)

## 說明

拿到一台新機器卻不知道如何設定？

一身程式功力無用武之地是非常難受的，這裡就來簡單創建新使用者與 ssh 開啟連線吧！

## 創建使用者

語法

```python
$ sudo useradd -s /path/to/shell -d /home/{dirname} -m -G {secondary-group} {username}
$ sudo passwd {username}
```

範例

```python
$ sudo useradd -s /bin/bash -d /home/ivankao/ -m -G sudo ivankao
$ sudo passwd ivankao
```

## 開啟帳號 ssh 連線

sudo vim `/etc/ssh/sshd_config`

```python
# PasswordAuthentication no
PasswordAuthentication yes
```

重啟

```python
sudo service ssh restart
```

## 開啟 root 帳號 ssh 連線

sudo vim `/etc/ssh/sshd_config`

```python
# PermitRootLogin no
PermitRootLogin yes
```

重啟

```python
sudo service ssh restart
```

設定 root 密碼

```python
sudo passwd root
```

## 參考資料

[https://www.cyberciti.biz/faq/create-a-user-account-on-ubuntu-linux/](https://www.cyberciti.biz/faq/create-a-user-account-on-ubuntu-linux/)

[https://serverpilot.io/docs/how-to-enable-ssh-password-authentication/](https://serverpilot.io/docs/how-to-enable-ssh-password-authentication/)