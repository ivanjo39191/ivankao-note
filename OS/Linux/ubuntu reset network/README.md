# ubuntu reset network

Created: February 1, 2022 5:22 PM
Tags: OS

```html
$ sudo nano /etc/default/grub
GRUB_CMDLINE_LINUX=""
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
$ sudo grub-mkconfig -o /boot/grub/grub.cfg
```

```html
sudo dhclient enp0s3
sudo dhclient eth1
sudo dhclient eth2
sudo dhclient eth3
```

[https://cometlc.pixnet.net/blog/post/5736664-修改預設網卡名稱eth0-on-ubuntu-18.04-16.04](https://cometlc.pixnet.net/blog/post/5736664-%E4%BF%AE%E6%94%B9%E9%A0%90%E8%A8%AD%E7%B6%B2%E5%8D%A1%E5%90%8D%E7%A8%B1eth0-on-ubuntu-18.04-16.04)

[https://blog.csdn.net/xiexf189/article/details/107357680](https://blog.csdn.net/xiexf189/article/details/107357680)