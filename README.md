# 大象同学的热带雨林 🐘

个人博客，基于 [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) Jekyll 主题搭建，托管在 GitHub Pages。

## 文章列表

- 搭建博客 & 工具链
  - [Minimal-Mistakes-Jekyll 搭建 GitHub 静态博客](https://daxmate.github.io/blog/jekyll-build-github-pages-static-website/) (2020-10-16)
  - [Git 笔记](https://daxmate.github.io/blog/git/) (2020-10-22)
  - [YAML 笔记](https://daxmate.github.io/blog/yaml/) (2020-11-17)
  - [Bash 脚本笔记](https://daxmate.github.io/blog/bash-scripting/) (2021-11-23)
- Python & 开发
  - [PyInstaller 打包 Python 应用](https://daxmate.github.io/blog/pyinstaller/) (2020-12-29)
  - [BeautifulSoup 笔记](https://daxmate.github.io/blog/beautifulsoup/) (2021-11-22)
  - [Vim 配置 C++ 开发环境](https://daxmate.github.io/blog/vim-settings-for-cpp-development/) (2022-03-21)
  - [设置 Karabiner 解决 Xcode Vim 模式问题](https://daxmate.github.io/blog/karabiner/) (2022-03-26)
- 环境 & 工具
  - [Rime 输入法配置](https://daxmate.github.io/blog/Rime/) (2020-10-18)
  - [Insomnia 使用笔记](https://daxmate.github.io/blog/insomnia/) (2021-02-14)
  - [Heroku 笔记](https://daxmate.github.io/blog/heroku/) (2021-02-26)
- 折腾记录
  - [腾讯云轻量应用服务器搭建 VPN](https://daxmate.github.io/blog/tencent-cloud/) (2021-11-21)
  - [重启](https://daxmate.github.io/blog/restart/) (2021-11-15)
- 其他
  - [Hello World](https://daxmate.github.io/blog/hello-world/) (2026-06-25)

## 本地开发

```bash
# 安装依赖
bundle install

# 本地预览
RUBYOPT="-Eutf-8" bundle exec jekyll serve --config _config.yml,_config.dev.yml

# 浏览器打开 http://localhost:4000
```

> 注：因 Ruby 3.4 兼容性问题，本地预览需用 `_config.dev.yml` 覆盖 `remote_theme` 为 Gem 版主题。推送到 GitHub Pages 时自动构建。

## 协议

© 大象同学
