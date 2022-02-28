# Linux sudo 免密碼登入

Created: February 1, 2022 5:22 PM
Tags: OS

修改sudusers文件

Centos 6

```
visudo 
#########################
root  ALL=(ALL)    ALL
# 添加使用者
username ALL=(ALL)  ALL
# 添加免密碼
username ALL=(ALL)  NOPASSWD:ALL

```

Ubuntu 20.04

如果你的使用者屬於 sudo 或 admin group 則須放在群組通用規則之後

```

visudo 
#########################
root    ALL=(ALL:ALL) ALL
# 添加使用者
ivankao ALL=(ALL:ALL)  ALL
# 添加免密碼
username ALL=(ALL:ALL)  NOPASSWD:ALL

```

[https://blog.csdn.net/Dream_angel_Z/article/details/45841109](https://blog.csdn.net/Dream_angel_Z/article/details/45841109)

[https://www.huaweicloud.com/articles/295a6c07155c0469fa0f9ca92864aa2b.html](https://www.huaweicloud.com/articles/295a6c07155c0469fa0f9ca92864aa2b.html)

[https://unix.stackexchange.com/questions/434889/why-does-my-setup-for-no-password-prompt-in-etc-sudoers-not-work](https://unix.stackexchange.com/questions/434889/why-does-my-setup-for-no-password-prompt-in-etc-sudoers-not-work)