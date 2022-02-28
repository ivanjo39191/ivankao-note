# git remove commit

Created: February 1, 2022 5:22 PM
Tags: Git

首先，刪除本地存儲庫上的提交。

你可以使用`git rebase -i`來做到這一點。

例如，如果它是你的最後一次提交，你可以執行`git rebase -i HEAD~2`並刪除編輯器窗口中的第二行。

然後，使用`git push origin branchName --force`強制推送到 GitHub

參考

[Git Magic Chapter 5: Lessons of History — And Then Some](http://www-cs-students.stanford.edu/~blynn/gitmagic/ch05.html#_8230_and_then_some)

[https://michael-hsu.medium.com/移除-github-上面的-commit-91d62ca424a1](https://michael-hsu.medium.com/%E7%A7%BB%E9%99%A4-github-%E4%B8%8A%E9%9D%A2%E7%9A%84-commit-91d62ca424a1)