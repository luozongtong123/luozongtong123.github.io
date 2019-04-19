
前天发现不蒜子网页访问计数一直是正在加载状态，没法正常工作。  

<!--more-->

{{% figure class="center" src="/img/busuanzi-domain/busuanzi.png" alt="不蒜子" title="不蒜子" %}}  

本站的访问量计数用的是<a href="https://busuanzi.ibruce.info//" rel="noopener" target="_blank">不蒜子</a>。  

even 主题启用不蒜子很简单，只需要在博客配置文件里启用即可。  

加载不蒜子脚本的语句在 `layouts/partials/head.html` 文件中。  

只需要将文件中的 `dn-lbstatics.qbox.me` 替换为 `busuanzi.ibruce.info` 即可修复。  




[![HitCount](http://hits.dwyl.io/ztluo/post.svg)](http://hits.dwyl.io/ztluo/post)

