在 YouTube 上看了个关于介绍 Visual Studio Code 的视频，这篇是笔记。
<!--more-->

{{% figure class="left" src="/img/vs-code-can-do-that/build_ppt.png" alt="build_ppt" title="Microsoft Build 2018 Keynote" %}}

视频是 Microsoft Build 2018 的录像。

# 视频

{{% youtube "x5GzCohd4eo" %}}

如果你无法访问 YouTube，B 站的搬运如下：
[VS Code Can Do That?! VS Code Tips and Tricks](https://www.bilibili.com/video/av34408553)

# 主题

You may know or may not know, VS Code 的主题非常丰富。视频中首先说了一下主题，
其中提到了 [Cobalt2](https://marketplace.visualstudio.com/items?itemName=wesbos.theme-cobalt2) 主题。这个主题是一个有助于保护眼睛的主题。

# 合字

这是一个非常炫酷的功能。Font Ligatures (合字) 可以将几个符号显示时变成一个符号，其效果如下：

{{% figure class="center" src="/img/vs-code-can-do-that/Fira-Code.png" alt="Fira Code" title="Fira Code" %}}

要实现这个功能首先需要到 GitHub 上下载安装 [Fira Code 字体](https://github.com/tonsky/FiraCode)。
下载后将压缩包解压，Windows 上，打开 ttf 文件夹，全选字体 > 右键 > 安装。  

VS Code 中将下面的语句添加到设置文件夹（可能需要重启才会生效）：

``` json
"editor.fontFamily": "'Fira Code'",
"editor.fontLigatures": true
```

需要注意的是，如果你需要在字体中选择多个字体，那么你需要将 `'Fira Code'` 放到第一的位置。

# 一些快捷键

<kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>P</kbd> ：调出命令面板;  
<kbd>Ctrl</kbd> + <kbd>P</kbd> + <kbd>P</kbd> ：在最近打开的文件之间切换;  
<kbd>Ctrl</kbd> + <kbd>B</kbd>：开关侧栏;  
<kbd>Ctrl</kbd> + <kbd>`</kbd>：开关终端;  
<kbd>Ctrl</kbd> + <kbd>J</kbd>：开关底栏;  

# 一些小技巧

<kbd>Ctrl</kbd> + <kbd>P</kbd> 然后输入 @ ，就可以在当前文件的大纲(Outline)范围内快速跳转，你甚至可以输入<kbd>：</kbd> 然后过滤你需要的内容;  
<kbd>Ctrl</kbd> + <kbd>P</kbd> 然后输入<kbd>：</kbd>可以快速跳转到特定行号；  
<kbd>Ctrl</kbd> + <kbd>P</kbd> 然后输入<kbd>?</kbd>可以查看帮助；  

# Emmet 和 Prettier formatter

Emmet (前身为 Zen Coding) 是一个能大幅度提高前端开发效率的一个工具。Emmet 可以自动补全 HTML和CSS等，
甚至可以直接计算表达式的值和自动填写图片的大小。  

Prettier formatter 是一个可以自动美化 JavaScript/TypeScript/CSS 的扩展。  

由于我不搞前端，这部分就不细说了。  

# 远程调试 Azure 上的 Node.js

很强大。(不懂 Node.js)

# 提到的插件

1. Hop Light ：Burke Holland 喜欢的颜色主题
2. Winter is Coming Theme：John Papa 喜欢的颜色主题
3. Material Icon Theme：提到的一个文件图标主题
4. Cobalt2 Theme Official：据说是对眼睛更好的一个颜色主题
5. Azure Cosmos DB：一个可以管理 MongoDB 数据库的扩展

>发现视频主要讲的是前端开发相关的。两位讲的很有趣。    

