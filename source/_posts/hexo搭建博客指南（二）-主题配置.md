---
title: hexo搭建博客指南（二）--主题配置
date: 2017-02-18 15:09:07
categories: hexo
tags:
- hexo
---
### 一 主题选择  

&emsp;&emsp;由于hexo默认的主题并不让人满意，因此要想搭建一个让自己心仪的博客你还需要一款好看的主题。hexo的主题你可以冲一下两个网站中获取：  
[https://github.com/hexojs/hexo/wiki/Themes](https://github.com/hexojs/hexo/wiki/Themes)  
[https://hexo.io/zh-cn/docs/themes.html](https://hexo.io/zh-cn/docs/themes.html)  


### 二 主题配置  

#### 2.1 安装主题

&emsp;&emsp;你可以选择自己心仪的主题，然后改一下hexo的配置文件_config.yml文件。  
``` bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: hexo-theme-air
``` 
你只需要将它改成你的主题名字  


#### 2.2 配置主题  
&emsp;&emsp;你需要了解一下hexo的主题配置文件_config.yml(注意与hexo根目录下的配置文件不是一个文件)的一些配置:
``` bash
menu:
  首页: /
  项目: /项目/
  职业生涯: /职业生涯/
  随感: /随感/
  归档: /archives
  关于我: /about

widgets:
- search
- category
- recent_posts
- tagcloud

```  
##### **menu**
&emsp;&emsp;下面的是你想要为你的网站划分的版块。前面是版块名称，后面是相应的路径。你可通过如下命令来新建新的版块页面：  
``` bash
hexo new page 版块名
```  
hexo会自动在source文件下建立相应的文件夹。   
**注意：**这里有坑。  

- 我曾新建完一个新的页面后却无法访问，究其原因是当我访问这个页面是浏览器的协议自动从https换成了http，结果找不到资源。原因未知，解决办法：将hexo中的东西都删掉，重搭。。。。。   
- 又遇一坑，当我将我的版块页面所在的文件夹取名为中文时，我需要在主题配置文件中的menu下相应的版块路径后面多家一个“/”，否则报错，而英文的就不需要，原因未知。
- 坑3，当使用md写博客时，网站默认显示的博文是居中的，可通过修改主题下的source/css/_partial/article.styl文件中blockquote解决。  
这些估计都是我选择的light主题中的坑。  

##### **widget**  
&emsp;&emsp;这下面的是一些网站侧边的一些小部件，比如我的网站右边的分类和标签云。你可以在你的主题下面的/layout/_widget下面找到你的主题支持哪些小部件，你只需要将这些部件名依照格式放在widget下即可显示在网站中。  

##### **categories 和 tags**
&emsp;&emsp;categories和tags是hexo内置的标签，你只需要在编写博客时加入以下配置：  
``` bash
---
title: hexo搭建博客指南（二）--主题配置
date: 2017-02-18 15:09:07
categories: hexo
tags:
- hexo
---
``` 
即可在widget中的category和tag中显示你的分类与标签，categories一般用于文章分类，你也可用它来生成你的版块页；tags一般则是用于为搜索引擎提供你这篇博文的关键字。  

### 三 总结

&emsp;&emsp;文章到这里也差不多说完了，hexo的其他知识点你可以通过google搜索或者研究hexo的官方文档[https://hexo.io/zh-cn/docs/asset-folders.html](https://hexo.io/zh-cn/docs/asset-folders.html)来获得。下篇我将介绍hexo的相关插件使用。