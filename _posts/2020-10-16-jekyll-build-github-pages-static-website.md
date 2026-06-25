---
hidden: true
layout: single
title: "Minimal-Mistakes-Jekyll搭建GitHub静态博客"
date: 2020-10-16
author: 'dax'
categories: "IT"
typora-root-url: ../../bestdax.github.io
---



## 为什么需要一个自己的空间

现在网上有很多可以写博客的地方，提供的功能也很丰富，为什么我们还要折腾自己弄一个GitHub上的空间呢？我主要原因有两个：

1. 自己弄这件事情够折腾😄️

2. 别人的地盘上有别人的规矩，老子不吃这一套。

   其实还可以加上第三点，自己弄，定制程度高，虽然会多花些时间和精力，但是可以做出真正自己喜欢的东西。配合`Git`可以对博客进行一次一次的修改，直到写出满意的内容为止。

   

   

## 准备工作

首先，第一步需要有一个GitHub账号。这个只要注册一下就可以。

然后，根据[GitHub Pages](https://pages.github.com/)上指示的步骤就可以写个`Hello World`的网页啦。

当然，不可能以后的博客都写成`html`格式的来提交，这就轮到`Jekyll`出场啦。

## 安装Jekyll

参照[Jekyll官方手册](https://jekyllrb.com/docs/installation/)来进行安装，下面是我在macOS上安装的过程。

1. 安装`homebrew`，这个有已经有了。然后就安装`ruby`

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   
   # Install Ruby
   brew install ruby
   ```

2. 注意，系统里面之前如果已经安装过ruby，但是不是用brew安装的话，可能会在后面出现报错。现在新安装的ruby应该是3.0以上的版本。可以在终端里面用`ruby -v`来看一下ruby的版本。如果不是3.0以上的版本，将下面命令添加到终端设置文件里面，我这里用的zsh，所以添加到`~/.zshrc`里面。

   ```bash
   export PATH=/usr/local/opt/ruby/bin:$PATH
   ```

3. 然后就可以安装bundle和jekyll了。

   ```bash
   sudo gem install bundle jekyll
   ```

   gem就类似Python里面的pip，bundle和jekyll类似库，bundle我觉得就是可以通过Gemfile来进行库的批量安装。

## 基于Minimal-Mistaks主题建Github静态网站

从这个主题的名字就明白设置里面可以少走很多弯路😭️，为什么我还是这么难……

[MM官方手册](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)里面提供了三种方法来安装主题，我试过后，下面这种操作适合我。

1. 从[MM主题仓库](https://github.com/mmistakes/minimal-mistakes/)Fork到自己的空间，然后改名为`my_github_username.github.io`，把第一个部分替换成自己的用户名。

2. 克隆到本地

   ```bash
   git clone https://github.com/my_github_username.github.io
   ```

3. 在终端里面进入这个新克隆的文件夹，然后运行如下命令

   ```bash
   bundle add webrick
   ```

   缺少这步的话会发生错误。

4. 然后需要进行一点细微的修改

   1. 网站根目录下新建一个_posts文件夹，用来放博客文章。
   2. 在assets文件夹下面新建一个images文件夹，用来放图片。
   3. 删除以下文件，因为不需要。
      1. `.editorconfig`
      2. `.gitattributes`
      3. `.github`
      4. `/docs`
      5. `/test`
      6. `CHANGELOG.md`
      7. `minimal-mistakes-jekyll.gemspec`
      8. `README.md`
      9. `screenshot-layouts.png`
      10. `screenshot.png`

这些步骤完成之后，个人博客的框架就搭建好了。可以用`bundle exec jekyll serve`来运行一个本地的服务器，然后在浏览器里面输入`127.0.0.1:4000`就可以看到效果了。如果满意的话，用Git推送到GitHub上就发布网站了。在`my_github_username.github.io`里就可以访问自己的博客网站了。



## 调整设置

设置的调整只要在`_config.yml`里进行就可以了。可以到GitHub里面找到别人的设置抄一下，必要的信息改成自己的就可以了。

1. 网站统计：

   ```yaml
   analytics:
     provider: "google-gtag"
     google:
       tracking_id: "UA-1234567-8"
       anonymize_ip: false # default
   ```

   需要自己去Google Analytics注册并生成一个tracking_id。

2. 评论功能：

   

3. 分类设置，

## 博文的格式

```
---
layout: single
title:  "Welcome to Jekyll!"
date:   2020-10-16 14:26:20 +0800
categories: "IT"
---
```

并且，**博客日志的文件名是有固定格式的**。`YEAR-MONTH-DAY-title.MARKUP`，其中，年份是4位数，月日都是两位数，标记的格式一般用`markdown`的话写`.md`就可以了。

## 配合Typora

用`Typora`配合书写`markdown`是一件很惬意的事情，最重要的是插入图片会非常方便。首先要把图片的文件夹给设置一下：

<img src="/assets/images/image-20201103153039481.png" alt="image-20201103153039481" style="zoom:50%;" />

然后还要设置图片的根目录，在`格式`→`图片`→`设置根目录`中把根目录设置为网站的根目录文件夹，这样以后插入的图片就能在GitHub上正确显示了。

