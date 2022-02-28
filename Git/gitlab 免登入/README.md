# gitlab 免登入

Created: February 1, 2022 5:22 PM
Tags: Git

[https://stackoverflow.com/questions/35942754/how-can-i-save-username-and-password-in-git](https://stackoverflow.com/questions/35942754/how-can-i-save-username-and-password-in-git)

```html
git config --global credential.helper store
```

帳密會被存在

```html
~/.git-credentials
```

建議使用 ssh key 較佳

[https://docs.gitlab.com/ee/ssh/](https://docs.gitlab.com/ee/ssh/)