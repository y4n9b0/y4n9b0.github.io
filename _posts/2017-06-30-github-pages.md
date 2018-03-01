---
layout: post
title:  "Github Pages"
date:   2017-06-30 17:00:00 +0800
categories: Tutorial
tags: Github&#160;Pages
published: true
---

* content
{:toc}


# Welcome
```
GitHub Pages is a static site hosting service and doesn't support server-side code such as, PHP, Ruby, or Python.
GitHub Pages is designed to host your personal, organization, or project pages directly from a GitHub repository.
```
Github Pages 是一个静态网站寄存服务,不支持服务端代码,比如PHP, Ruby, 或者 Python.  
Github Pages 的页面基于Github仓库.

# Sample
首先你要有个[Github](https://github.com/)账号.什么?搞软件还没有Github账号:scream:!你要不要考虑下回家种红薯呀 亲:blush:?  

注册了Github账号之后,创建一个名为username.github.io的repository.  
友提:这里的username是你的Github账号的username,并不是指"username"这个固定的字符串.  

创建好之后克隆下来依次执行以下操作:
```bash
git clone https://github.com/username/username.github.io
cd username.github.io
echo "Hello Lisa" > index.html
git add --all
git commit -m "Initial commit of my first blog"
git push -u origin master
```
Github Pages博客就搭好了,然后从浏览器进入  
<https://username.github.io/>  
就可以看见  
![/styles/images/githubpages/hellolisa.png]({{ '/styles/images/githubpages/hellolisa.png' | prepend: site.baseurl  }})  

这是一个没有任何技术含量的博客示例,只有一个页面,只有一句问候.  
虽然她很丑,可是她很温柔.后面我们再慢慢让她变漂亮:princess:.

# Advanced
事实上你在Github的每一个仓库都会对应一个Github Pages地址(repositoryName是仓库名):  
<https://username.github.io/repositoryName/>  
username.github.io这个仓库对应地址有点例外,它对应的Github Pages地址就是我们上面所说的  
<https://username.github.io/>  

一个仓库对应着一个Github Pages博客的源码,不同点在于username.github.io这个仓库的Github Pages地址是在仓库创建的时候默认开通绑定  
而其他任何仓库都需要自己去手动开通,对比上图:  
![/styles/images/githubpages/repository1step2hell.png]({{ '/styles/images/githubpages/repository1step2hell.png' | prepend: site.baseurl  }} "repository 1step2hell")  
<br>
![/styles/images/githubpages/repositoryLisa.png]({{ '/styles/images/githubpages/repositoryLisa.png' | prepend: site.baseurl  }} "repository Lisa")  

<br>
开通也很简单,打开Github想要开通的仓库,找到仓库的settings,下拉找到Github Pages,把None换为选择master branch,然后点击save即可  
![/styles/images/githubpages/openRepositoryLisaGithubPages.png]({{ '/styles/images/githubpages/openRepositoryLisaGithubPages.png' | prepend: site.baseurl  }})  

<br>
然后再用浏览器打开仓库对应的地址  
<https://username.github.io/repositoryName/>  
![/styles/images/githubpages/imLisa.png]({{ '/styles/images/githubpages/imLisa.png' | prepend: site.baseurl  }} "I'm Lisa")  

<br>
还有一个诀窍：在仓库上增加"gh-pages"分支，便自动开通该仓库项目的Github Pages地址。与上面的区别在于这种方式默认博客地址访问的是"gh-pages"分支的内容。

# Issue
写到这里,其实有一个问题,根据规则,新的博客对应的地址其实是在默认开通绑定的博客下一级.那么如果默认博客里边本身就有个页面,并且地址和新博客地址一样,这下同一个地址对应两个页面,那么会怎样呢:open_mouth:?  

实践出真知,测她一扳手.  
为了便于测试,我在1step2hell.github.io仓库里找一个现有的下级页面,点击右上角的About->Author进入author页面,这个页面的地址为<https://1step2hell.github.io/author/>,完美地符合了冲突测试条件.
于是新建一个叫做author的仓库(注意大小写必须和上面地址里保持一致),这个仓库对应的Github Pages地址也应该为<https://1step2hell.github.io/author/>.输入这个url的时候该出现哪个页面呢?是1step2hell.github.io仓库里author下的index.html?还是author仓库里的index.html?   
见证奇迹的时刻到了,下注买马的赶紧了,买定不离手.  
从浏览器打开<https://1step2hell.github.io/author/>,结果是这个地址会对应author仓库里的页面.猜对了吗:sunglasses:?  

Github Pages在解析博客主域名下一级路劲的时候会优先去查找是否有同名的仓库,没有同名的仓库才会去默认的仓库里找是否有对应的页面.  
当然这里还涉及到author是否开通绑定,是否有README,是否有404.html等.  
以上只是我基于测试结果的一个看起来合理的猜测,因为这是Github Pages内部的机制.砂锅打破底,写了封邮件去确认下.有回复了再来更新.  
希望会有回复,上次给Google的chromium team写邮件请教问题,结果根本没人鸟.是我英语太烂了?或许吧:pensive:  

最后,附上Github Pages的相关[Github Help](https://help.github.com/categories/github-pages-basics/)
