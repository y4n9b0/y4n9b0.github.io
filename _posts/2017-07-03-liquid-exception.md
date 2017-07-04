---
layout: post
title:  "Liquid Exception"
date:   2017-07-03 13:00:00 +0800
categories: Issue
tags: Liquid
published: true
---

* content
{:toc}


# Issue
在写Jekyll Directory structure的_includes时,使用了{% raw %} {% include file.ext %} {% endraw %}这个字符串作为示例,介绍如何包含_includes下的文件.  
提交后收到了Github的编译报错邮件  
```
The page build failed for the `master` branch with the following error:
A file was included in `_posts/2017-06-30-blog.md` that is a symlink or does not exist in your `_includes` directory. For more information, see
https://help.github.com/articles/page-build-failed-file-is-a-symlink/.

For information on troubleshooting Jekyll see:
https://help.github.com/articles/troubleshooting-jekyll-builds

If you have any questions you can contact us by replying to this email.
```
查看提示<https://help.github.com/articles/page-build-failed-file-is-a-symlink/>,发现错误原因是Jekyll以为我要在文章里包含_includes下的file.ext文件,但是这个文件却不存在,从而编译失败.  

事实上,这里我只是想举例表达{% raw %} {% include file.ext %} {% endraw %}这个字符串,并不想去调用file.ext这个文件,很明显Jekyll使用的Liquid把这个特定格式的字符串给转换了.  

试了很多方法,无论是标记成行内还是区块都不行.在{和%之间加空格倒是可以解决,但是这样我想表达的效果就变了,如果有人复制过去作为代码使用就会出问题.  

最后搜索到了这个[issue](https://github.com/imathis/octopress/issues/466),[samselikoff](https://github.com/samselikoff)的方法完美解决了问题.  
在文章里这样写  
&#123;&#37; raw &#37;&#125;
  &#123;&#37; include file.ext &#37;&#125;
&#123;&#37; endraw &#37;&#125;   
liquid转换后就成了{% raw %} {% include file.ext %} {% endraw %}.  

但是这里又有个问题,在写这篇文章的时候我没办法把  
&#123;&#37; raw &#37;&#125;
  &#123;&#123; include file.ext &#125;&#125;
&#123;&#37; endraw &#37;&#125;  
作为raw text显示出来,  
因为第一个&#123;&#37; endraw &#37;&#125;
总是和&#123;&#37; raw &#37;&#125;)配对去了,没办法把&#123;&#37; endraw &#37;&#125;本身作为raw text给予显示.  

在请教了Liquid写文档的小哥[Adam Hollett](https://github.com/admhlt)之后得到了答案把{}和%用HTML实体替换掉就可以了.  
{ 对应为 &amp;#123;  
% 对应为 &amp;#37;  
} 对应为 &amp;#125;  

写到这里又有个问题:laughing:,&amp;在Markdown里属于特殊字符,如果想显示也需要用HTML实体&amp;amp;替换掉<https://daringfireball.net/projects/markdown/syntax#autoescape>  

所以你看到的这句话  
&#8195;&#8195;{ 对应为 &amp;#123;  
其实在我写的Markdown文件里是这样子的  
&#8195;&#8195;{ 对应为 &amp;amp#123;

而你看到的这句话  
&#8195;&#8195;{ 对应为 &amp;amp;#123;  
对应的Markdown又是这样子的  
&#8195;&#8195;{ 对应为 &amp;amp;amp;#123;  

而你看到的这句话  
&#8195;&#8195;{ 对应为 &amp;amp;amp;#123;  
对应的Markdown...  

因为&amp;对应的HTML实体&amp;amp;开头又包含自身,所以咱们可以无限玩下去  
疯了有木有:joy:
