# Access denied for root user in MySQL

Created: February 1, 2022 5:22 PM
Tags: Mysql

原因為 root 帳號有設置權限，一般帳號無法登入

sudo su 

mysql -u root -p 

```python
GRANT ALL on *.* to 'username'@'localhost' identified by 'password';
```

開啟權限後即可用一般的 linux user 透過 root 登入 mysql 

[https://stackoverflow.com/questions/11760177/access-denied-for-root-user-in-mysql-command-line](https://stackoverflow.com/questions/11760177/access-denied-for-root-user-in-mysql-command-line)