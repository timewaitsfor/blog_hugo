---
title: 使用手册：Git
date: 2020-09-15T11:23:04+08:00
categories: [使用手册]
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

# 上传全新的git到GitHub的repository

create a new repository on the command line
```sh
echo "# pro_lab" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:timewaitsfor/pro_lab.git
git push -u origin main
```
# 将已有的代码的git到GitHub的repository

push an existing repository from the command line
```sh
git remote add origin git@github.com:timewaitsfor/pro_lab.git
git branch -M main
git push -u origin main
```