---
title: 使用Hexo和github pages搭建博客
date: 2016-04-18 19:50:26
updated: 2016-04-18 19:50:26
categories: blog
tags: [blogs, hexo]   
---

## 1. 简介

hexo是一个基于node.js的静态博客程序，可以方便的生成静态网页（纯html）支持多个平台（Windows/MAC/Linux），风格优雅，更适合写技术博客，与hexo类似的博客程序还有jekyll，jekyll被github着力推荐官方就提供了jekyll教程，但是jekyll是基于ruby写的，并且关于代码高亮没找到比较好的方案，就选择了用hexo

<!--more-->

## 2. 配置环境
### 2.1 安装git
作者用的是mac，可以使用[brew](http://brew.sh/)下面命令安装

```shell
$ brew install git
```

也可以直接上[git官网](https://git-scm.com/download/)下载安装

### 2.2 安装node.js
同样的，mac可以使用brew安装，新版的node.js已经包含npm工具，不需要再另外安装了

```shell
$ brew install node
```

可以通过下面命令检查是否已安装

```shell
$ node -v
$ npm -v
```

如果是windows用户可以通过官网下载 [jode.js](http://nodejs.org)

### 2.3 Hexo安装
上面的安装完成后，接下来安装hexo

``` shell
npm install hexo-cli -g   #-g表示全局安装, npm默认为当前项目安装
hexo init blog            #在当前目录下新建blog目录初始化博客
cd blog                   #进入blog目录
#npm install               
hexo generate             #根据当前配置生成静态页面
hexo server               #启动本地服务，默认为：[http://localhost:4000/](http://localhost:4000/)
```

接下来就可以通过[http://localhost:4000/](http://localhost:4000/)查看效果了
![hello hexo](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-18/1497950.jpg)

## 3. 配置github pages
每个github账户都可以有一个外部空间/Responsitory，可以直接通过`用户名.github.io`访问到该仓库的内容
  * 在github上新增一个responsitory，仓库名为 `用户名.github.io` 或 `用户名.github.com`
  * 创建完成后，github会自动将 用户名.github.io指向该仓库，默认访问根目录下的`index.html`页面
  * 可以进入Responsitory的Setting页查看

github会提供几个模板搭建站点，我们可以不用他提供的模板，可以在仓库里面，添加一个简单的index.html文件，如果能通过`用户名.github.com`访问，则表明创建成功了

## 4. 写博客
hexo的文章存放在source目录下

```bash
├── source  
|   ├── _posts    #存放文章  
|   └── _drafts   #存放草稿
```

```shell
$ hexo new post "postName"        # 在source/_posts 目录下创建postName.md文件
```

创建文件的命名格式可以在_config.yml文件配置
```yml
# Writing
new_post_name: :year-:month-:day-:title.md
```

文件创建完成后会自动生成以下格式（可以自己添加）
```
---
title: 使用Hexo和github pages搭建博客
date: 2016-04-18 19:50:26
categories: blog                  # 分类
tags: [blogs, hexo]               # 标签，格式：[标签, 标签2]
---
```

关与写作的各种参数可以参见：[https://hexo.io/docs/writing.html](https://hexo.io/docs/writing.html)

写完后预览的时候发现，文章在首页就全部显示出来了，如果不想全部显示，可以在文章中间添加下面标记，在首页列表就会出现`Read More`的标记

```html
<!--more-->
```

Hexo支持使用Markdown语法写文章，我比较习惯用Atom写Markdown，Atom有个hexo插件


## 5. 主题
官方自带主题基本够用，有能力可以自己改造，当然，网上已经有很多人做了一些很好看的主题了，我们可以直接拿来用，下面是官方列出的一些主题，找到喜欢的可以直接用

> [https://github.com/hexojs/hexo/wiki/Themes](https://github.com/hexojs/hexo/wiki/Themes)
> [https://hexo.io/themes](https://hexo.io/themes/)

在hexo上，主题放在themes目录下，我们只需要把别人做好的主题clone下来就好了，然后在`_config.yml`修改一下配置
例如：我们可以[https://github.com/xiangming/landscape-plus](https://github.com/xiangming/landscape-plus)这个主题clone下来

```git
git clone git@github.com:xiangming/landscape-plus.git themes/landscape-plus
```

修改设置`_config.yml`

```yml
theme: landscape-plus
```
![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-18/53534541.jpg)


## 6. 添加多说评论插件
到[多说官网](http://duoshuo.com/)注册和创建一个站点

修改配置
到`themes/landscape-plus/_config.yml`添加多说的配置，shortname即注册的站点名称

```yml
# Duoshuo
duoshuo_shortname: bomo
```

参见官方说明，替换评论相关的代码[http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)

完成，如下图评论有了
![评论](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-18/61189510.jpg)

## 7. 部署到github上
修改配置`_config.yml`

``` yml
deploy:
  type: git
  repository: https://github.com/zhengbomo/zhengbomo.github.io.git
  branch: master
```

安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)

```shell
$ npm install hexo-deployer-git --save
```

部署hexo到git上

```shell
$ hexo deploy
```

部署过程需要输入账号密码，然后会push到github上，参考：[https://hexo.io/docs/deployment.html](https://hexo.io/docs/deployment.html)

> hexo部署时会把最终生成的博客文件（public目录下的文件）push到git远程仓库，而博客程序还是在本地，当我们切换电脑的时候，无法对博客进行重新编辑和发布，这个时候我们可以在git添加一个分支`hexo`用来存放博客程序和编写的内容，详情可以参见： [git创建分支hexo存放博客程序](/2016-04-19/hexo-branch/)

## 8. 域名绑定
> 通常域名在[godaddy](https://www.godaddy.com/)注册比较靠谱，这个是最大的域名提供商，而且不需要备案，支持支付宝付款，购买的时候可以使用优惠码会便宜一些，网上有很多优惠码，可以自行搜索，购买过程很简单，这里就不贴了

1. 注册和配置DNS服务器
[Godaddy](https://www.godaddy.com/)自带的域名解析服务器比较慢，在国内推荐使用[DNSpod](https://www.dnspod.cn/)：快，免费，稳定。
  * 到DNSpod注册登陆，然后到用户中心，添加域名，例如我的域名为`bomobox.org`
  ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/12170815.jpg)

  * 进入设置
  ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/72132532.jpg)
    添加两个A记录指向github提供的ip，参见[这里](https://help.github.com/articles/setting-up-an-apex-domain/)
    ```
    192.30.252.153
    192.30.252.154
    ```

    添加一个CNAME记录指向自己的github域名：`username.github.io`  
    把其他的删除

2. 注册域名和配置DNS
  * 到[Godaddy](https://www.godaddy.com/)购买域名完成后完成后进入[MyAccount](https://mya.godaddy.com/)
  ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/12984808.jpg)

  * 进入`DNS Manager`修改DNS服务器
    ```
    f1g1ns1.dnspod.net
    f1g1ns2.dnspod.net
    ```
    ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/60235109.jpg)

3. 到github仓库的根目录添加CNAME文件，文件内添加自己的域名，否则会出现404访问错误，也可以在hexo的`source`目录下添加，然后不熟到github
  ![](http://7xqzvt.com1.z0.glb.clouddn.com/16-4-29/76622961.jpg)

上面步骤设置完成后可能会有几个小时的延迟，才能生效，总的来说还是比较简单的

## 9. 问题
在使用别人的主题的时候可能会报错或者有些功能用不了，原因可能是部分插件没有安装，例如RSS用不了，那可能是`hexo-generator-feed`没安装，下面列举一些常用的插件，建议都安装，没有用到也没有关系，需要先到hexo程序目录下在执行命令，插件位于`node_modules`目录下

```shell
  $ npm install hexo-generator-feed --save                  #支持RSS
  $ npm install hexo-generator-sitemap --save               #生成站点地图
  $ npm install hexo-generator-baidu-sitemap --save         #生成百度站点地图
  $ npm install hexo-html-minifier --save                   #HTML 压缩
  $ npm install hexo-uglify --save                          #JavaScript 压缩
  $ npm install hexo-clean-css --save                       #CSS 压缩插件
  $ npm install hexo-generator-seo-friendly-sitemap --save  #SEO优化
  $ npm install hexo-deployer-git --save                    #git部署插件

```
并在博客配置文件`_config.yml`配置plugin
```yml
Plugins:
- hexo-generator-feed
- hexo-generator-sitemap
```

> 更多插件可以在[https://hexo.io/plugins/](https://hexo.io/plugins/)找到

## 10. Atom插件
由于我编写md使用的是Atom，这里推荐几个Atom上的插件

* [markdown-scroll-sync](https://atom.io/packages/markdown-scroll-sync)：Markdown预览实时滚动，自带的预览不支持实时滚动
* [markdown-writer](https://atom.io/packages/markdown-writer)：Markdown协作工具
* [Date](https://atom.io/packages/date)：快速插入当前时间的工具
* [atom-hexo](https://atom.io/packages/atom-hexo)：快速添加draft，post，publish，deploy

## 11. 总结
使用hexo搭建博客环境还是非常方便的，基本上都是自动的，当然还有一些详细的配置，例如分页，分类，评论等，Hexo支持的插件也相当多的，接下来可以好好写博客了，以后再慢慢完善了，今天先到这里


## 12. 参考链接
> [https://hexo.io](https://hexo.io)
