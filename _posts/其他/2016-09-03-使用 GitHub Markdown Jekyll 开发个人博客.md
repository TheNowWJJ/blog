---
layout: post
title: 使用 GitHub Markdown Jekyll 开发个人博客
date: 2016-09-03 01:08:00 +0800
categories: 其他
tag: jekyll markdown github
---

## GitHub简介

Git是一个分布式的版本控制系统，最初由Linus Torvalds编写，用作Linux内核代码的管理。在推出后，Git在其它项目中也取得了很大成功，尤其是在Ruby社区中。目前，包括Rubinius、Merb和Bitcoin在内的很多知名项目都使用了Git。Git同样可以被诸如Capistrano和Vlad the Deployer这样的部署工具所使用。

> [百度百科:GitHub简介](http://baike.baidu
.com/link?url=sjR5Bsqi5H4FkplgzTJWtdreJ86zA1pyx4Mp_0JdbrvBrMn6u_9618irNxNpzq-Tnp4LE4l31QdvijxdKZZu7a)

> [如何高效利用GitHub](http://www.yangzhiping.com/tech/github.html)

> [git - 简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)

## Markdown简介

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。
Markdown具有一系列衍生版本，用于扩展Markdown的功能（如表格、脚注、内嵌HTML等等），这些功能原初的Markdown尚不具备，它们能让Markdown转换成更多的格式，例如LaTeX，Docbook。Markdown增强版中比较有名的有Markdown Extra、MultiMarkdown、 Maruku等。这些衍生版本要么基于工具，如Pandoc；要么基于网站，如GitHub和Wikipedia，在语法上基本兼容，但在一些语法和渲染效果上有改动。

> [百度百科:Markdown简介](http://baike.baidu.com/link?url=_2GpTyKb0kBrVlGDOcGvmBjTDSPhazbeHDGeBghfVYbndvW94rERblhCJfazQZ9-VCL7iINFDbpMZWbL1Y956a)

> [Markdown——入门指南](http://www.jianshu.com/p/1e402922ee32/)

> [Markdown 语法说明 (简体中文版)](http://www.appinn.com/markdown/)

## Jekyll简介

jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。

> [百度百科:Jekyll简介](http://baike.baidu.com/link?url=frJ6Wu9gTPEIDJ8V-YvX94eUfcHRc_H2mNAPzr2CHviW1dBRpN-EivNxFdRCA6vzyhFm5y50miqiQPDnCCAFFK)

> [Jekyll中文文档](http://jekyllcn.com/)

## 好处

我们可以像提交代码一样来管理我们的博客,简单方便快捷,而且还免费. 在此感谢[GitHub](www.github.com)!

## 搭建本地开发环境
本文开发环境基于Ubuntu Kylin 16.04 LTS

* 安装Ruby

  ```bash
    sudo apt-get install ruby
  ```

* 安装gem

  这个按照官网的教程安装上就可以了

  > [gem官网](https://rubygems.org/pages/download)

* 使用gem安装jekyll

  ```bash
   sudo gem install jekyll
  ```

## 选择Jekyll themes并clone到本地
  我选择的是 > [Jekyll theme链接:](https://github.com/liungkejin/liungkejin.github.io)

* 点击github 的 fork功能 将代码fork到自己的仓库
* git clone 刚刚fork的仓库到本地

  ```bash
    cd /home/barton/develop/code/git/
    git clone https://github.com/sunshineasbefore/liungkejin.github.io.git
  ```
* 创建自己的博客文件夹

  ```bash
    mkdir blog
  ```
* 复制`liungkejin.github.io`到`blog`

  ```bash
    cp -R liungkejin.github.io/* ./blog
  ```

* 本地调试

  ```bash
    cd /home/barton/develop/code/git/blog
    jekyll serve #不是jekyll server
  ```

  等待运行完成后,在控制台也会打印出URL,复制此URL用浏览器打开即可访问,比如输入localhost:4000.
  **注意此处可能也会输入localhost:4000/blog 这个要根据_config.yml文件中是否配置baseurl来决定**

## push本地blog文件夹到github仓库

* 首先需要在github上创建blog仓库
* 然后在`/home/barton/develop/code/git/blog`文件夹下执行一下命令:

  ```bash
    git init #初始化git 本地仓库
    git remote set-url origin https://github.com/sunshineasbefore/blog.git #更改本地仓库的远程仓库地址
    git add ./* #添加文件到git版本管理中
    git commit -m "init" #将变更提交到本地仓库
    git branch gh-pages #创建新的gh-pages分支(如果要在github中使用pages功能需要这么一个分支)
    git push -u origin gh-pages #推送本地gh-pages分支到远程仓库,此步骤需要输入github用户名(邮箱)和密码
  ```

## 在blog仓库中设置开启pages功能
  网上介绍这个功能的文章有很多,咱就不再重复一便了...
  进行到此步骤,在设置中就可以看到github分配给你的域名,比如说:sunshineasbefore.github.io/blog
  点击此域名即可访问使用github pages功能搭建的静态博客.

## 博客配置

* Jekyll的语法以及目录结构请参考Jekyll的中文文档.(文章开头有)
* 更改_config.yml

  更改后内容如下:

  ```yml
  # Site settings
  title: 阳光如初.
  description: http://veryjava.cn/
  favicon: /assets/img/avatar.JPG
  baseurl: #此处写 `/` + `仓库名称`,如果使用个人名,则不用写
  url: http://veryjava.cn/ # 如果没有个人域名(稍后介绍),此处写 github分配给你的域名,比如说:sunshineasbefore.github.io/blog
  rss_url:

  # Build settings

  highlighter: pygments # 语法高亮

  timezone: Asia/Shanghai

  markdown: kramdown # 此处使用kramdown编辑器
  kramdown:
    input: GFM # 扩展语法.这个很重要,如果不写,则语法高亮不起作用
    auto_ids: true
    auto_id_prefix: 'id-'
  ```
* 更改作者设置和博客设置
  这两个文件主要是个人简介功能和博客头部个人签名设置
  这两个文件在_data目录下.
  更改后author.yml和blog.yml内容如下:
  author.yml:

  ```yml
  # Author settings
  name: 王
  title: Java 开发工程师
  address: 山东, 济南
  email: work_wjj@163.com
  github: sunshineasbefore
  gavatar: /assets/img/avatar.JPG
  workHistory:
  - work3:
    company: 济南三际电子商务有限公司
    location: 山东, 济南
    title: Java 高级开发工程师
    started: 2015
    duration: (2015.8 - 至今)
    description: Linux服务器维护,项目开发,参与系统架构设计

  - work2:
    company: 上海某科技公司
    location: 中国，上海
    title: Java 开发工程师
    started: 2014
    duration: (2014.4 - 2015.8)
    description: 在平安好车做新车购,C2C项目

  - work1:
    company: 济南某对日外包企业
    location: 山东, 济南
    title: Java 程序员
    started: 2013
    duration: (2012.6 - 2014.4)
    description: 对日项目外包

  educationHistory:
  - education1:
    organization: 烟台某大学
    degree: 学士
    major: 软件技术
    started: 2008
    duration: (2008.9 - 2012.7)
    description: C/C++, Java, C#, 等
  languages:
  - language1:
    name: 中文
    proficiency: 母语
  - language2:
    name: 英语
    proficiency: 有限
  programmingSkills:
  - Java:
    name: Java
    percentage: 90%
  - Bash:
    name: Bash Shell
    percentage: 60%
  - JS:
    name: JavaScript
    percentage: 70%
  - Docker:
    name: Docker
    percentage: 50%
  - git:
    name: Git
    percentage: 70%
  - MySQL:
    name: MySQL
    percentage: 70%

  ```

  blog.yml:

  ```yml
  title: 阳光如初.
  description: 怀揣梦想,永不止步!
  ```

## 书写博客
* 格式
  在_posts文件夹下新建文档,格式如下:`yyyy-mm-dd-文件名称.md` (文件名称可以带空格)
  jekyll以日期+名称的格式命名文章,否则将不识别.
* 文档头部

  ```
  ---
  layout: post
  title:  "Welcome to Jekyll!"
  date:   2014-01-27 21:57:11
  categories: jekyll update
  ---
  ```

  其中:

  * title：文章的标题
  * date：文章的日期
  * categories：定义了文章所属的目录，一个list，将会根据这个目录的list来创建目录并将文章html放在生成的目录下，文章分类时候用，这里就不使用了
  * layout：文章所使用的模板名称，也就是_layouts中定义的模板的文件名去掉.html
  * tags：例子中没有，定义了文章的标签，也是一个list，文章分类时候用，这里就不使用了
* kramdown
  kramdown遵循大多数markdown的语法,但是有一些细节的地方不太一致.比如说代码块和之前的文字描述之间必须有空行;Tab键和空格的表现形式差距很大等,
* push文章到仓库

  将写好的文章push到远程仓库的gh-pages分支后,点击github分配给你的域名便可访问了.

## 个性域名设置.
* 申请域名:
  我是从[万网](https://wanwang.aliyun.com/) 申请的域名veryjava.cn. (已经归属于阿里云旗下)
  具体申请步骤不再介绍.
* 两种方式绑定个性域名:
  * 使用CNAME文件.
    在博客文件夹的根目录创建CNAME文件.

    ```bash
    touch CNAME
    ```

    编辑CNAME文件放置自己的个性域名:

    ```bash
     vim CNAME
    ```

    编辑后的内容:

    ```
    veryjava.cn
    ```

    是的,仅仅就只有一行,并且不要带www

    然后将CNAME文件push到远程仓库就可以了.当然不要忘记设置[域名的DNS解析](http://jingyan.baidu
    .com/article/3c343ff70fb6e60d3779632f.html).
  * 直接在仓库的设置选项中设置
    打开仓库设置选项 找到`Custom domain`功能,添加自己的域名即可.
* 需要注意的地方
  DNS解析到国外需要24小时的时间,所以设置好后,不要在看到仓库设置中的黄色警告时着急.等等明天就好了.
