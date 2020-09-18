---
title: "Git使用手册"
date: 2020-09-15T11:23:04+08:00
---

# abstract

# .gitignore 文件的使用

参考：
[廖雪峰-忽略特殊文件](https://www.liaoxuefeng.com/wiki/896043488029600/900004590234208)
[编程猴子-gitignore 不起作用的解决办法](https://www.cnblogs.com/sumg/p/10251247.html)

```
git rm -r --cached .
```

# 上传文件

```sh
git status
git add .
git commit -m ""
git push -u origin master
```

如果出现不好上传的情况：（贺老师git指导）
```sh
git add .
git commit
git pull --rebase
git push origin master
```