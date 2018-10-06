使用 Git 的 Submodule 来管理 hugo 静态博客非常方便。   

<!--more-->

> 使用前提：   
>
> 已经在 Gituhb 上创建了 `username.github.io` 的仓库，可以正常使用 hugo 生成静态博客文件，使用 Git 上传到 Github Pages 后可以正常创建博客。

## 初次使用

首先，在 Github 上创建一个用于存放博客根目录的仓库。 将仓库 clone 到合适的位置。把博客根目录中的所有文件拷贝到本地仓库中。   

```bash
git clone https://github.com/username/yourrepo.git
```

**注：如果你在添加 hugo 主题的时候用的是 Submodule ，那么下一步可以略过。**   

以将主题添加为 Submodule ，如果之前添加过主题需要删除整个主题文件夹。

```bash
# 在博客根目录下
git submodule add https://github.com/username/themename.git themes/themename
```



然后，将 `public` 文件夹删除，把 用于创建 Github Pages 的仓库以 `Submodule` 的形式添加到本地仓库。

```bash
git submodule add https://github.com/username/username.github.io.git public
# 这行命令会将博客静态文件仓库克隆到本地的 public 文件夹，并同时将该仓库添加为附仓库的子模块
```

运行 `git status` 后可以发现本地仓库的根目录中多了一个名为 `.gitmodules` 的文件，这个文件用于存放和子模块相关的的配置。   

## 使用脚本部署博客

接下来当我们需要更新部署博客的时候只需要运行下面的 bat 脚本即可。

```bash
::cmd 
::deploy.bat 
@echo off 
hugo 

:: public 
echo. 
echo cd to public... 
cd public 

echo. 
echo add all files... 
git add . 

echo. 
echo print status of Git Submodle... 
git status 

echo. 
echo commit files... 
git commit -m "update blog" 

echo. 
echo push Submodle to GitHub... 
git push 

:: father repo 
echo. 
echo back to repo... 
cd .. 

echo. 
echo add all files... 
git add . 

echo. 
echo print status of Git repo... 
git status 

echo. 
echo commit files... 
git commit -m "update blog" 

echo. 
echo push repo to GitHub... 
git push 

:: finsh 
echo. 
echo deploy finished. 

```

hugo 生成的文件是 LF 换行符，git 添加文件的时候会显示将 LF 替换为 CRLF 的警告，使用如下命令可以关闭这个警告。

```bash
git config --global core.safecrlf false
```



## 在另一台电脑上使用

当我们需要在一台新的电脑上更新博客的时候只需要将 Github 上博客的仓库递归克隆下来即可。

```bash
git clone https://github.com/username/yourblog.git yourblog --recursive 
cd yourblog/public 
git checkout master 
```

注意：更新子模块后 Git 检出项目的指定版本，但是不在分支内。为了跟踪变更你需要检出 `master`

分支，或者创建一个新的分支，为了简化操作我们直接检出 `master` 分支。<sup>[[1]](#ref01)</sup>



## 关于子模块

子模块的应用场景是这样的：项目一依赖项目二，项目一由你来维护，项目二可能不由你维护或者由你维护但是你想保持两个项目的相互的独立性。这时你可以将项目二添加为项目一的子模块。   

Git 子模块的一些常规操作：<sup>[[2]](#ref02)</sup><sup>[[3]](#ref03)</sup>

```bash
# 添加子模块
git submodule add https://github.com/username/gitname.git gitname 

# 递归克隆带有子模块的仓库
git clone https://github.com/username/yourblog.git yourblog --recursive 
# 上面递归克隆的命令等同于下面几行代码
git clone https://github.com/username/yourblog.git yourblog 
git submodule init    # 初始化子模块
git submodule update  # 更新子模块
# 此处要注意，如果你需要对子模块做出更改，需要创建或检出一个分支，Git默认不在任何分支上

# 删除子模块
# 逆初始化模块，其中{MOD_NAME}为模块目录，执行后可发现模块目录被清空
git submodule deinit {MOD_NAME} 
# 删除.gitmodules中记录的模块信息（--cached选项清除.git/modules中的缓存）
git rm --cached {MOD_NAME} 
# 提交更改到代码库，可观察到'.gitmodules'内容发生变更
git commit -am "Remove a submodule."

# 修改某子模块的远程库(url)
# 首先手动修改'.gitmodules'文件中对应模块的”url“属性 
# 更新子模块的 url 
git submodule sync 
# 提交变更 
git commit -am "Update submodule url." 
```





## 参考资料

1. <a id="ref01">[Git 工具 - 子模块](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)</a>
2. <a id="ref02">[Git submodule 子模块的管理和使用](https://www.jianshu.com/p/9000cd49822c)</a>
3. <a id="ref02">[如何优雅的删除子模块 (submodule) 或修改 Submodule URL](https://www.jianshu.com/p/ed0cb6c75e25)</a>
