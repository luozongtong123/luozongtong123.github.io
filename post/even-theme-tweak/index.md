当前博客的主题是 [even](https://github.com/zt-luo/hugo-theme-even)。
为了满足我刁钻的口味，有一些主题的小细节需要修改一下。折腾过程做了点总结。
<!--more-->  

# 修改主题颜色

在 `src\css\_variables.scss` 文件中添加需要的颜色，并通过修改颜色配置修改的主题颜色。

``` diff
diff --git "a/..\\Temp\\TortoiseGit\\_variables-2c7773b.000.scss" "b/..\\even\\src\\css\\_variables.scss"
index 06d7507..39bdf23 100644
--- "a/..\\Temp\\TortoiseGit\\_variables-2c7773b.000.scss"
+++ "b/..\\even\\src\\css\\_variables.scss"
@@ -4,14 +4,15 @@
 
 // ========== Theme Color ========== //
 // Config here to change theme color
-// Default | Mint Green | Cobalt Blue | Hot Pink | Dark Violet
-$theme-color-config: 'Default';
+// Default | Mint Green | Cobalt Blue | luo Blue | Hot Pink | Dark Violet
+$theme-color-config: 'luo Blue';
 
 // Default theme color map
 $theme-color-map: (
   'Default': #c05b4d #f8f5ec,
   'Mint Green': #16982B #f5f5f5,
   'Cobalt Blue': #0047AB #f0f2f5,
+  'luo Blue': #2980B9 #f0f2f5,
   'Hot Pink': #FF69B4 #f8f5f5,
   'Dark Violet': #9932CC #f5f4fa
 );
```

# 使用 Fira Code 字体

前文有提到过这个字体，这是一个支持连字的等宽字体，非常适合码代码。  

还是修改 `src\css\_variables.scss` 文件。 

{{% admonition note %}}

此处只是将默认字体改为 `Fira Code`，如果用户的电脑上没有安装这个字体的话就会使用其他备选的字体。

{{% /admonition %}}

``` diff
@@ -193,7 +206,7 @@ $code-color: #c7254e !default;
 $code-font-size: 13px !default;
 
 // Font family of the code.
-$code-font-family: Consolas, Monaco, Menlo, Consolas, monospace !default;
+$code-font-family: 'Fira Code', 'Consolas', 'Monaco', 'Menlo', monospace !default;
 
 // Color of code highlight, solarized.
 $code-highlight-color: (
```

接下来我们需要将字体文件打包到网站上。  

首先修改 `src\css\_variables.scss` ，添加 `Fira Code` 字体家族。

``` diff
@@ -91,11 +92,23 @@ $header-padding: 20px 20px !default;
   font-style: normal;
 }
 
+// Font family: Fira Code
+@font-face {
+  font-family: 'Fira Code';
+  src: url('../fonts/FiraCode/FiraCode-Regular.eot');
+  src: local('Fira Code'), url('../fonts/FiraCode/FiraCode-Regular.eot?#iefix') format('embedded-opentype'),
+  url('../fonts/FiraCode/FiraCode-Regular.woff2') format('woff2'),
+  url('../fonts/FiraCode/FiraCode-Regular.woff') format('woff'),
+  url('../fonts/FiraCode/FiraCode-Regular.ttf') format('truetype');
+  font-weight: lighter;
+  font-style: normal;
+}
+
 // Font size of the logo.
 $logo-font-size: 48px !default;
```

然后，修改 `src\\webpack.config.js` 将字体文件打包到 `static` 文件夹。

``` diff 
diff --git "a/..\\Temp\\TortoiseGit\\webpack.config-eb3dafd.000.js" "b/..\\even\\src\\webpack.config.js"
index f2218be..d96c24a 100644
--- "..\\Temp\\TortoiseGit\\webpack.config-eb3dafd.000.js"
+++ "..\\even\\src\\webpack.config.js"
@@ -42,6 +42,10 @@ module.exports = {
       {
         test: /apple-chancery-webfont\.(woff|woff2|eot|ttf|otf|svg)$/,
         use: ['file-loader?name=[path][name].[ext]']
+      },
+      {
+        test: /FiraCode-Regular\.(woff|woff2|eot|ttf|otf)$/,
+        use: ['file-loader?name=[path][name].[ext]']
       }
     ]
   },
```

# 完善代码高亮配置

原作主题的代码高亮只启用了一小部分，接下来添加一些我需要的代码高亮，去掉一些不需要的高亮，同时把使用的 `highlightjs` 升级一下。  

到官网的[下载](https://highlightjs.org/download/)页面，勾选需要的高亮的语言，下载。。。  
替换掉 `static\lib\highlight\highlight.pack.js`  文件。

修改 `src\css\_variables.scss` 文件中的 `$code-type-list`。

``` scss
// Code type list.
$code-type-list: (
  // Custom code type
  language-bash: "Bash",
  language-sh: "sh",
  language-c: "C",
  language-cs: "C#",
  language-cpp: "C++",
  language-css: "CSS",
  language-diff: "Diff",
  language-patch: "Patch",
  language-html: "HTML",
  language-xml: "XML",
  language-json: "JSON",
  language-js: "JavaScript",
  language-javascript: "JavaScript",
  language-makefile: "Makefile",
  language-markdown: "Markdown",
  language-python: "Python",
  language-sql: "SQL",
  language-shell: "Shell",

  language-asm: "ARM Assembly",
  language-arduino: "Arduino",
  language-autohotkey: "AutoHotkey",
  language-cmake: "CMake",
  language-cmd: "cmd",
  language-bat: "bat",
  language-docker: "Docker",
  language-fsharp: "F#",
  language-matlab: "Matlab",
  language-powershell: "PowerShell",
  language-scss: "Scss",
  language-tex: "TeX",
  language-twig: "Twig",
  language-vb: "VB",
  language-yml: "YAML",
  language-yaml: "YAML",
  language-toml: "TOML"
) !default;
```

改用 `highlightjs` 官方默认的 CSS 样式渲染。将 `default.css` 文件中的内容复制到 `src\css\_partial\_post\_code.scss` 中的如下位置。

``` scss
    .code {
      // 复制到这
    }
```

# 更新 Gitalk

主题自带的 Gitalk 版本较低，不支持评论预览，更新到最新版。  

到 [Github](https://github.com/gitalk/gitalk/tree/master/dist) 上下载最新的 CSS 和 JS 文件，
替换 `static\lib\gitalk` 里的文件，并相应修改 `layouts\partials\comments.html` 中的语句。

``` diff
 {{- else -}}
-  <link rel="stylesheet" href="{{ "lib/gitalk/gitalk-1.2.2.min.css" | relURL }}">
-  <script src="{{ "lib/gitalk/gitalk-1.2.2.min.js" | relURL }}"></script>
+  <link rel="stylesheet" href="{{ "lib/gitalk/gitalk.css" | relURL }}">
+  <script src="{{ "lib/gitalk/gitalk.min.js" | relURL }}"></script>
 {{- end }}
```

# 添加 admonition

在浏览 [olOwOlo](https://blog.olowolo.com/) 的[博文](https://blog.olowolo.com/post/generate-json-api-and-graphql-api-by-elide-and-spring/)时候发现，博客中的 `admonition`  非常漂亮。在博文下面询问是怎么实现的，无奈作者好像消失了一样。所以只能自己实现一套了。看了一下 Hugo 的[模版系统](https://gohugo.io/templates/introduction/)，抄了[olOwOlo](https://blog.olowolo.com/) 的 `CSS`，实现了如下的 `admonition` 。自我感觉效果还不错。   

具体实现就不详细讲了，想要了解的话直接去看我的[代码](https://github.com/zt-luo/hugo-theme-even/commit/24840c633e9fb54a188012965d62ae94e087fa71)就行。  
想要使用的直接 Fork 我的[仓库](https://github.com/zt-luo/hugo-theme-even)即可。

{{% admonition tip %}}

这是一个 `admonition` 示例的第一行。  
这是一个 `admonition` 示例的第二行。  

{{% /admonition %}}


# Build

{{% admonition attention %}}
上述所有修改 `src` 文件夹下文件的操作都需要重新编译才能实际生效。  
{{% /admonition %}}

``` powershell
# 使用 scoop 安装 Node.js LTS 
scoop install nodejs-lts
# 查看 Node.js 版本号
node -v
# 当前版本为 v8.12.0 (2018.10.28)
# 注意目前 (2018.10.28) 只能使用版本号最高为 10 的 Node.js
cd src
# 安装依赖
npm install
# 编译文件
npm run build
```

# admonition 演示

tip
{{% admonition tip %}}

这是一个 `admonition` 示例的第一行。  
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

note、hint
{{% admonition note %}}

这是一个 `admonition` 示例的第一行。   
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

warning、important
{{% admonition warning %}}

这是一个 `admonition` 示例的第一行。   
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

question
{{% admonition question %}}

这是一个 `admonition` 示例的第一行。   
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

conclusion
{{% admonition conclusion %}}

这是一个 `admonition` 示例的第一行。   
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

bullshit
{{% admonition bullshit %}}

这是一个 `admonition` 示例的第一行。   
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

attention、caution、danger、error
{{% admonition caution %}}

这是一个 `admonition` 示例的第一行。   
这是一个 `admonition` 示例的第二行。  

这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行，这是一个长行。

{{% /admonition %}}

