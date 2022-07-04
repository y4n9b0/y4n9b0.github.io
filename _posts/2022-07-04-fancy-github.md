---
layout: post
title:  Fancy Github
date:   2022-07-04 10:00:00 +0800
categories: publish
tags: github
published: true
---

* content
{:toc}

## Github Pages

关于 github pages 我在 17 年就讲过了，不再重述 [https://y4n9b0.github.io/2017/06/30/github-pages/](https://y4n9b0.github.io/2017/06/30/github-pages/){:target="_blank"}

唯一需要补充的一点便是 github pages source 的选择：<br>
![github pages source]({{ '/styles/images/fancygithub/github_pages_source.jpg' | prepend: site.baseurl }}){:width="700" height="auto"}  

## Github Profile

创建一个与 github 账号同名的仓库，该仓库便是你 github 首页个人资料 repository。有点 github 彩蛋的意思。<br>
仓库里新建一个 README.md 文件，可以在里边作一些自我吹捧，利用 github api 接口查询 github status 等等。<br>
网上有大把玩得很花的 profile repository，自行搜索、模仿、创新。

## Steam Status

接下来是今天的重头戏，教大家如何在 github 账号首页显示个人 steam 账号下所有游戏状态。<br>
众所周知，github 和 steam 的用户群体存在高度重合，而 steam 又对外提供了 API 接口。
于是各路大神又开始脑洞大开，首先创建一个 github gist，并在 github 账号首页 pin 该条 gist；然后建一个仓库，该仓库存放访问 steam status 相关的脚本，
脚本获取的结果输出到最开始创建的 gist，最后在该仓库下创建一个 action 用于定期执行脚本。

上效果图：<br>
![github pin gist]({{ '/styles/images/fancygithub/github_pin_gist.jpg' | prepend: site.baseurl }}){:width="400" height="auto"}  

### 准备工作

* 创建一个 public 的 gist [https://gist.github.com/](https://gist.github.com/){:target="_blank"}，<br>
  创建好的 gist url 链接格式 https://gist.github.com/`${github account}`/`${gist id}`，记下 gist id。<br>
  比如 https://gist.github.com/y4n9b0/f8576ab794e13a9196a2e8f1de843544，gist id 为 f8576ab794e13a9196a2e8f1de843544。
* 创建一个拥有 gist 权限的 github token [https://github.com/settings/tokens/new](https://github.com/settings/tokens/new){:target="_blank"}
  * Expiration 选择 No expiration
  * Select Scopes 勾选上 gist<br>
  生成好的 token 一定要记得复制保存，这玩意儿一旦不复制后面就再也见不着了
* 创建 steam API key 
  * [https://steamcommunity.com/dev/registerkey](https://steamcommunity.com/dev/registerkey){:target="_blank"}
  * [https://steamcommunity.com/dev/apikey](https://steamcommunity.com/dev/apikey){:target="_blank"}
* 查询 steam 账号的 64 位 id，登录 steam 社区，profiles 页面 url 最后一串数字即是 https://steamcommunity.com/profiles/${steamID64}，
  * [https://steamcommunity.com](https://steamcommunity.com){:target="_blank"}
  * [https://steamid.io](https://steamid.io){:target="_blank"}
* 查找 steam 游戏 app id，可以在对应游戏的 steam 商店的 url 获取到游戏 id，如 https://store.steampowered.com/app/`${app id}`/CounterStrike_Global_Offensive/
  * csgo appId 730
  * dota2 appId 570

### pin gist

将刚才创建的 gist pin 在 github 账号首页，操作极其简单，自行搜索

### 创建仓库

* fork 仓库 [https://github.com/y4n9b0/steam](https://github.com/y4n9b0/steam){:target="_blank"}
* 添加仓库 secrets **Settings** > **Secrets**，https://github.com/`${github account}`/steam/settings/secrets/actions/new
  * GIST_ID，准备工作中记下的 gist id
  * GH_TOKEN，准备工作中生成并复制的 github token
  * STEAM_ID，准备工作中的 steam 账号 64 位 id
  * STEAM_API_KEY，准备工作中创建的 steam API 密钥
  * APP_ID，准备工作中查询的 app id，多个 app id 之间用英文逗号分隔，比如 730, 570

### 定期任务 action

https://github.com/`${github account}`/steam/blob/master/.github/workflows/schedule.yml 文件里指定了 action，
使用 [cron](https://en.wikipedia.org/wiki/Cron){:target="_blank"} tag，可自行调整查询更新 gist 周期。

需要注意的是 github action 定时任务触发时机不是精确的，大量的 github action 任务都设置在整点，导致 github 在整点的 action 非常多，而平时非常少，
所以 github 使用了任务队列来执行定时 action（不包括手动触发的 action）。
[https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule){:target="_blank"}

github action 执行有日期上限，许多人仓库不再使用后就放任不管，为了避免浪费资源继续在这些没意义的仓库执行 action，github 设置了 action 执行期限，
目前貌似仓库没任何动作之后 90 天便会停止 action，在停止之前 github 会给对应的账号发邮件提醒，若还需继续执行 action，只需按邮件重新触发一下即可。

**Note**<br>
由于不可描述的原因，以上诸多链接大概率无法直接访问，各位请小心谨慎，以免犯了破坏计算机系统、网络系统等**主义特色罪

## cloud disk

把 github 仓库当作云盘，存放各种资源。需要注意两点
* 单个文件大小不能超过 100M，事实上单个文件超过 50M，github 便会提醒使用 Large File Storage(LFS)
* 不要把任何机密上传 github，即便后续通过 force push 删除也无济于事。github 并不会真正删除对应的 commit，通过 hash id 依然可以在 github 网页访问。
  * [https://docs.github.com/cn/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository](https://docs.github.com/cn/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository){:target="_blank"}
  * [https://stackoverflow.com/questions/872565/remove-sensitive-files-and-their-commits-from-git-history](https://stackoverflow.com/questions/872565/remove-sensitive-files-and-their-commits-from-git-history){:target="_blank"}
