# Linux 查找檔案中的字串數量

Created: February 22, 2022 3:35 PM
Tags: Linux, OS

```bash
awk -v RS="@#$j" '{print gsub(/targetStr/,"&")}' filename
awk  '{s+=gsub(/targetStr/,"&")}END{print s}' filename
```

[https://www.itread01.com/content/1545553863.html](https://www.itread01.com/content/1545553863.html)