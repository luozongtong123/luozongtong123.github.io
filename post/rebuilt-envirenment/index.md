在 cloud9 上写博客。
<!--more-->
## 新建一个 workspace

在 [cloud9](https://c9.io) 官网注册登录完毕后。点击左侧绿色按钮新建一个 workspace。选择 Private to the people 、 Hosted 、 Node.js 。

## 新建博客目录

到根目录新建一个博客目录
```
cd 
mkdir myblog
```

## 安装配置 Hexo 程序

依次运行如下命令

```
npm install hexo-cli -g
npm install hexo --save
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
hexo init
cd themes/
git clone git@github.com:zt-luo/luocman.git
cd luocman/
cp _config-index.yml ~/myblog/
cd
cd myblog/
mv _config-index.yml _config.yml
rm -rf source/
git clone git@github.com:zt-luo/source.git
rm -rf scaffolds/
cd themes/luocman/
mv scaffolds/ ~/myblog/
npm install
npm install hexo-renderer-marked --save
npm install hexo-renderer-stylus --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
```



[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

