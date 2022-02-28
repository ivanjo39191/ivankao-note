# Git 儲存帳號密碼

Created: February 1, 2022 5:22 PM
Tags: Git

```bash
git config --global credential.helper store
```

如果我們 git clone 的下載程式碼的時候是連線的https://而不是git@git (ssh)的形式，當我們操作 git pull/push 到遠端的時候，總是提示我們輸入賬號和密碼才能操作成功，頻繁的輸入賬號和密碼會很麻煩。

# **1. 本地儲存帳號密碼**

git bash 進入你的專案目錄，輸入： git config --global credential.helper store

然後你會在你本地生成一個文字，上邊記錄你的賬號和密碼。當然這些你可以不用關心。 然後你使用上述的命令配置好之後，再操作一次 git pull，然後它會提示你輸入賬號密碼，這一次之後就不需要再次輸入密碼了。

# **2. 使用 SSH 連線**

1. Git Bash 進入 ssh 目錄

```
cd ~/.ssh
複製程式碼
```

1. 生成 SSH key (檔名：id_rsa, id_rsa.pub)

```
 ssh-keygen -t rsa -C "xxxxxx@yy.com"  #建議填寫自己真實有效的郵箱地址
複製程式碼
```

1. 文字編輯器開啟公鑰 `id_rsa.pub` 複製內容，新增到 Github setting。
2. 測試

```
ssh -T git@github.com
複製程式碼
```

> You've successfully authenticated, but GitHub does not provide shell access.
> 

**說明配置成功**

[https://iter01.com/196584.html](https://iter01.com/196584.html)