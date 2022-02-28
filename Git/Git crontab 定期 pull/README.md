# Git crontab 定期 pull

Created: February 1, 2022 5:22 PM
Tags: Git, OS

需先儲存git使用者資訊

 git config --global credential.helper store

並且 pull 過一次後才可免登入

```bash
*/1 * * * * cd /path/src && /usr/local/git/bin/git pull > /path/gitpull.log
```

切換目錄 cd /path/src

執行 git pull (可用 which git 查找 git 路徑)

寫入 log 檔位置 /path/gitpull.log