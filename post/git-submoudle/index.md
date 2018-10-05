使用 Git 的 Submoudle 来管理 hugo 静态博客非常方便。

<!--more-->

首先，在 Github 上创建两个用于存放博客内容的版本库。 

```
::cmd 
::deploy.bat 
@echo off 
hugo 
echo. 
echo print status of Git repo... 
git status 
cd public 
echo. 
echo print status of Git Submodle... 
git status 
echo. 
echo add all files... 
git add . 
echo. 
echo commit files... 
git commit -m "update blog" 
echo. 
echo push Submodle to Github... 
git push 
echo. 
echo back to repo... 
cd .. 
echo. 
echo print status of Git repo... 
git status 
echo. 
echo add all files... 
git add . 
echo. 
echo commit files... 
git commit -m "update blog" 
echo. 
echo push repo to Github... 
git push 

```

hugo 生成的文件是 LF 换行符，git 添加文件的时候会显示将 LF 替换为 CRLF 的警告，使用如下命令可以关闭这个警告。

```bash
git config --global core.safecrlf false
```


