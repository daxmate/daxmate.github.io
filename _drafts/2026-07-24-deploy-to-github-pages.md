---
layout: single
title:  "把前端项目部署到 GitHub Pages — 从一次真实动手聊聊"
date:   2026-07-24 09:10:00 +0800
categories:
  - Web
tags:
  - GitHub Pages
  - 部署
  - CI/CD
---

## 起因

前阵子写了一个数独 PWA，手机上玩没问题，但只要关掉本地的开发服务器，浏览器就打不开了。Service Worker 离线缓存配得好好的，为什么不行？

查了一下原因很简单：**手机通过局域网 HTTP 访问开发服务器，浏览器不允许在这种 origin 上注册 Service Worker。**

Chrome 的规则是：
- `http://localhost:5175` → 可以（开发模式豁免）
- `https://yourdomain.com` → 可以
- `http://192.168.x.x:5175` → 不行

没 HTTPS，Service Worker 根本装不上，离线能力自然无从谈起。

解决方案也很直接——要么上 HTTPS，要么找个现成的托管服务。我选了后者：GitHub Pages。

---

## 第一步：设置 base 路径

GitHub Pages 有两种托管方式：

- **用户/组织站点**：`https://username.github.io` — 根目录部署，资源路径从 `/` 开始
- **项目站点**：`https://username.github.io/repo-name/` — 子目录部署，路径需要加前缀

数独项目仓库叫 `sudoku`，所以最终地址是 `https://daxmate.github.io/sudoku/`。这意味着所有静态资源路径必须带 `/sudoku/` 前缀。

Vite 项目只需要加一行配置：

```js
export default defineConfig({
  base: '/sudoku/',  // 👈 加上这一行
  plugins: [vue(), VitePWA({ ... })],
})
```

这点很容易漏。很多人部署上去发现样式全乱、JS 报 404，十有八九是这个 `base` 没设对。

PWA 的 manifest 里的图标路径也得改——`/icon.svg` → `/sudoku/icon.svg`，否则图标加载不出来。

---

## 第二步：写 GitHub Actions workflow

GitHub Pages 在 2022 年改过版，现在不再推荐推 `gh-pages` 分支了，而是用 GitHub Actions 来构建和部署。好处是构建过程在 CI 里完成，本地不用留着构建产物。

我的 workflow 文件长这样：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npm run build
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist
      - uses: actions/deploy-pages@v4
```

几个要点：

**`permissions` 段**不能省。`actions/deploy-pages` 需要 `pages: write` 和 `id-token: write` 权限。

**`concurrency`** 保证连续推送时，上一个还在跑的 workflow 会被取消，不会排队堆积。

**`npm ci` 和 `npm install` 的区别**：前者会严格按照 `package-lock.json` 安装，不更新 lock 文件，构建环境更一致。CI 里优先用 `npm ci`。

**Node 版本**——GitHub Actions 的 runner 默认用 Node.js 20，但在 2025 年底 Node 20 已被标记为 deprecated。2026 年的环境默认是 Node 24，但运行 Node 20 的 action 会强制降级，触发警告。所以直接指定 `node-version: 22`（或 24）就行。

---

## 第三步：在仓库设置里开启 Pages

workflow 写好了、代码也推送了，然后 Actions 跑出来这个错误：

```
HttpError: Not Found
Get Pages site failed. Please verify that the repository has Pages enabled
```

别慌——不是 workflow 写错了，只是 GitHub Pages 还没在仓库设置里开启。

去仓库的 **Settings → Pages → Build and deployment → Source**，选 **GitHub Actions**。这一步主要是告诉 GitHub：你的站会通过某个 workflow 部署，请允许相应的 API 操作。

选完之后，再跑一次 workflow 就过了。

> 选完「GitHub Actions」后，GitHub 会推荐一些 starter workflow（Static HTML、Jekyll 等）。不用选，你推送在仓库里的 workflow 会自动生效。

---

## 几步之外：部署后的检查

如果一切顺利，访问 `https://daxmate.github.io/sudoku/` 就能看到页面了。

手机上打开这个 HTTPS 地址，Service Worker 正常注册。加到主屏幕后，关掉网络也能离线玩——当初折腾 HTTPS 的目标终于达到了。

不过 GitHub Pages 默认绑定的域名在国内访问速度不太理想。如果你也遇到这个问题，有几个选项：

- **套 Cloudflare CDN**：域名托管到 Cloudflare，会加速 GitHub Pages 的全球访问
- **迁移到 Cloudflare Pages**：用法一样，国内有节点，速度快很多
- **用国产的 Zeabur**：深圳团队做的，支持从 GitHub 仓库一键导入，国内访问无压力
- **或者就用 Gitee Pages**：免费版需要手动点击「部署」按钮，不如 CI 自动化方便

我目前还是用 GitHub Pages，毕竟和 Jekyll 博客在同一套体系里，省心。真遇到国内用户访问慢的时候再说吧。

---

## 回顾

这个部署过程看起来步骤不多，但每一步都有坑：
- `base` 路径没设对，资源全 404
- 忘了在 Settings 里开启 Pages，workflow 跑不过
- Node 版本太旧被 deprecated，CI 里报了 warning
- 不知道选「GitHub Actions」而不是「Deploy from a branch」的新版流程

回过头看，GitHub Pages + GitHub Actions 这套组合确实是部署前端项目最省心的方案之一。写一次 workflow，以后每次 `git push` 自动构建上线，比手动上传 FTP、配 Nginx 轻松多了。

而且免费、有 HTTPS、和代码仓库深度绑定——对个人项目和小型工具来说，够好了。
