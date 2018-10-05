使用 Git 的 Submoudle 来管理 hugo 静态博客非常方便。

<!--more-->

首先，初始化一个空的版本库，

```
# cmd
# deploy.bat
@echo off 
echo "print status of Git repo..." 
git status 
cd public 
echo "print status of Git Submodle..." 
git status 
echo "add all files..." 
git add . 
echo "commit files..." 
git commit -m "update blog" 
echo "push Submodle to Github..." 
git push 
echo "back to repo..." 
cd .. 
echo "print status of Git repo..." 
git status 
echo "add all files..." 
git add . 
echo "commit files..." 
git commit -m "update blog" 
echo "push repo to Github..." 
git push 

```


