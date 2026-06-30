# 大象同学的热带雨林 🐘

我的个人博客，基于 Jekyll + Minimal Mistakes 主题，托管在 GitHub Pages。

## 内容方向

- **从零开始学命令行** — 面向初学者的命令行系列教程
- **工具与效率** — macOS 工具、CLI 工具、自动化
- **编程笔记** — Python、C++、Vim 等

## 本地运行

```bash
bundle install
bundle exec jekyll serve --livereload
```

修改 `_config.yml` 后需重启，配合 [entr](https://eradman.com/entrproject/) 可自动重启：

```bash
echo _config.yml | entr -r bundle exec jekyll serve --livereload
```

## 发布流程

草稿放在 `_drafts/` 目录，不会发布。完成后移到 `_posts/`，按 `YYYY-MM-DD-slug.md` 命名。

```bash
# 发布草稿
git mv _drafts/my-post.md _posts/2026-01-01-my-post.md
```

## 许可证

博客内容 © 大象同学。代码部分基于 Minimal Mistakes 主题的 MIT 许可证。
