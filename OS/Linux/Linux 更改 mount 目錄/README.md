# Linux 更改 mount 目錄

Created: February 22, 2022 1:48 PM
Tags: Linux, OS

原本 /dev/mapper/mydata mount 在 /data

```
/dev/mapper/mydata  196G   50G  137G   27% /data
```

要將 mount 的 /data 改為 /opt

### umount

```
umount /data
```

出現該目錄正在忙碌的錯誤 ****is busy****

找出正在忙碌的原因

```bash
fuser -m /data

# /data: 2895920c
```

代表 process 2895920(pid) 有使用到此目錄, 後面 c 代表的意思可參考下述:

- c: current directory.
- e: executable being run.
- f: open file. f is omitted in default display mode.
- F: open file for writing. F is omitted in default display mode.
- r: root directory.
- m: mmap'ed file or shared library.

[https://blog.longwin.com.tw/20ㄝㄝ08/11/debian-ubuntu-linux-umount-device-busy-2008/](https://blog.longwin.com.tw/2008/11/debian-ubuntu-linux-umount-device-busy-2008/)

將使用到的 pid 結束後，重新 umount 即可

### mount

創建新目錄

```bash
mkdir /opt
```

mount

```bash
mount /dev/mapper/mydata /opt
```