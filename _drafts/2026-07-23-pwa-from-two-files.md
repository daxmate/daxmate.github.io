---
layout: single
title:  "给网页加 PWA：从两个文件开始"
date:   2026-07-23 18:27:00 +0800
categories:
  - Programming
tags:
  - PWA
  - Service Worker
  - manifest.json
  - 前端
  - 数独
---

> 本文用数独项目里的两个文件（`manifest.json` 和 `sw.js`）来介绍 PWA 的核心概念。读完你可以给自己网站加上离线缓存和安装能力。

## PWA 是什么

PWA（Progressive Web App）让一个普通网页具备类似原生 App 的能力：

- **离线可用** — 断网了也能打开
- **安装到桌面** — 像 App 一样有个图标
- **启动画面** — 短暂的加载体验

实现这些不需要学新框架，只需要一个配置文件 + 一个脚本。下面逐个拆解。

## 第一个文件：manifest.json

```json
{
  "name": "数独",
  "short_name": "数独",
  "description": "逻辑 · 专注 · 挑战",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4f46e5",
  "icons": [
    {
      "src": "/icon.svg",
      "sizes": "any",
      "type": "image/svg+xml",
      "purpose": "any maskable"
    }
  ]
}
```

每个字段讲清楚：

**`name` 和 `short_name`**
- `name` 是应用的全名，安装提示时显示
- `short_name` 是安装到桌面后图标下方的文字，空间有限时使用
- 一般 `short_name` 控制在 12 个字符以内

**`start_url`**
- 用户从桌面图标打开时加载的页面地址
- 通常是 `/`，如果你的应用装在子目录（比如 GitHub Pages 的 `username.github.io/project/`），要写成对应的路径

**`display`**
控制打开后的界面形式，有四个选项：

| 值 | 效果 |
|----|------|
| `fullscreen` | 全屏，不显示浏览器任何 UI |
| `standalone` | 像原生 App 一样，有标题栏但没有地址栏 |
| `minimal-ui` | 显示少量浏览器控件 |
| `browser` | 普通浏览器标签页（默认） |

数独游戏用 `standalone` 是最合适的——看起来像个正经 App。

**`background_color` 和 `theme_color`**
- `background_color`：启动时的背景色，在图标点下去到页面渲染出来之间会闪一下
- `theme_color`：浏览器地址栏、任务切换器的颜色
- 两个颜色可以不同。数独项目里把它们设成不一样——启动闪白色，主题用紫色

**`icons`**
安装到桌面用的图标。可以指定多个尺寸，系统选最合适的。数独用了一个 SVG，尺寸写 `"any"` 表示自适应。

> 传统做法要准备多个尺寸的 PNG（192x192、512x512），但现代浏览器已经支持 SVG 了，一个文件搞定所有尺寸。如果你的目标用户包含旧版本浏览器，建议同时放一个 192x192 的 PNG。

写完后，在 `index.html` 的 `<head>` 里引用它：

```html
<link rel="manifest" href="/manifest.json" />
```

## 第二个文件：sw.js（Service Worker）

Service Worker 是 PWA 的核心。可以把它理解成一个**运行在浏览器后台的代理**，你的网页发出去的请求都在它这里过一手——它可以决定从网络拿、从缓存拿、还是自己去造一个响应。

数独项目的 `sw.js` 有 30 行，分成三段生命周期。

### install 阶段——预缓存

```javascript
self.addEventListener('install', (e) => {
  e.waitUntil(
    caches.open(CACHE).then(c => c.addAll(FILES))
  )
  self.skipWaiting()
})
```

用户第一次访问你的页面时，Service Worker 被安装。`install` 事件里做的事：

1. `caches.open('sudoku-v1')` — 打开名叫 `sudoku-v1` 的缓存空间
2. `c.addAll(FILES)` — 把指定的文件全部下载并存入缓存
3. `self.skipWaiting()` — 立即激活，不等页面关闭

`FILES` 数组就是你要提前缓存的东西：

```javascript
const FILES = [
  '/',
  '/index.html',
  '/manifest.json',
  '/icon.svg',
]
```

这里的路径相对于网站的根目录。对于 Vite 项目，把构建物路径写进去就行。

### activate 阶段——清理旧缓存

```javascript
self.addEventListener('activate', (e) => {
  e.waitUntil(
    caches.keys().then(keys => Promise.all(
      keys.filter(k => k !== CACHE).map(k => caches.delete(k))
    ))
  )
})
```

当你更新了 Service Worker（比如把 `CACHE` 从 `'sudoku-v1'` 改成 `'sudoku-v2'`），新版本安装后需要清理旧缓存。这段代码做的就是：把所有不是当前版本的缓存删掉。

这样你发布新版本时，用户拿到的就是最新缓存的数据，不会混着旧版本的文件。

### fetch 阶段——响应请求

```javascript
self.addEventListener('fetch', (e) => {
  if (e.request.mode === 'navigate') {
    e.respondWith(
      fetch(e.request).catch(() => caches.match('/'))
    )
    return
  }
  e.respondWith(
    caches.match(e.request).then(r => r || fetch(e.request))
  )
})
```

每次页面发起请求（加载 JS、CSS、图片、AJAX 等）都会触发 `fetch` 事件。这里分了两种情况：

**导航请求（`navigate`）**：用户访问一个页面时。策略是**先走网络**，网络不行就返回缓存的首页。这就是 PWA "离线也能打开" 的核心——断网了也能看你上次缓存的页面。

**其他请求**：JS、CSS、图片等。策略是**缓存优先**：先在缓存里找，有就直接返回（快），没有再去网络拿。

这个策略叫 "stale-while-revalidate" 的简化版——牺牲一点实时性换取速度和离线能力。对于一个数独游戏（数据都在前端生成，不需要网络），这个策略是完美的。

## 第三部分：注册 Service Worker

单靠 `sw.js` 文件放在服务器上是不会起作用的——你需要告诉浏览器去注册它。在 `src/main.js`：

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
  })
}
```

`navigator.serviceWorker.register('/sw.js')` 告诉浏览器：帮我装上这个 Service Worker。

为什么要加 `if` 判断？老浏览器不支持 Service Worker，不做判断直接调用会报错。页面的功能不受影响，只是少了离线能力。

`window.addEventListener('load', ...)` 在页面完全加载后再注册——不阻塞首次渲染。

## 第四部分：iOS 的额外配置

PWA 在 Android 上支持得很好，iOS 上一直不太积极。不过最近几个版本 Safari 也跟进了。为了让 PWA 在 iPhone 上更像 App，`index.html` 里还加了这几行：

```html
<link rel="apple-touch-icon" href="/icon.svg" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<meta name="theme-color" content="#4f46e5" />
```

- `apple-touch-icon`：添加到主屏幕时用的图标
- `apple-mobile-web-app-capable`：告诉 Safari 这个页面可以全屏运行
- `apple-mobile-web-app-status-bar-style`：控制状态栏样式
- `theme-color`：Android 和桌面版 Chrome 会读，iOS 从 iOS 15 开始也支持

## 完整的 index.html head

数独项目的 `<head>` 完整内容长这样：

```html
<link rel="icon" type="image/svg+xml" href="/icon.svg" />
<link rel="apple-touch-icon" href="/icon.svg" />
<link rel="manifest" href="/manifest.json" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<meta name="theme-color" content="#4f46e5" />
```

## 怎么验证 PWA 是否生效

以 Chrome 为例：

1. 打开开发者工具（F12）→ **Application** 面板
2. 左边栏看到 **Manifest**：能看到 `manifest.json` 解析后的信息
3. **Service Workers**：能看到注册状态，这里也有 `skipWaiting` 和更新按钮
4. **Cache Storage**：能看到 `sudoku-v1` 这个缓存空间里有哪些文件

测试离线功能：

6. 开发者工具 → **Network** 面板 → 勾上 **Offline**
7. 刷新页面——如果能正常显示，说明 Service Worker 的缓存正常工作

手机上测试：用 HTTP 服务器访问（Vite 的 `npm run dev` 或者 `python3 -m http.server`）。有些功能在开发服务器上可能会受限，最好用生产构建测试：

```bash
npm run build    # Vite 构建
npx serve dist   # 启动静态服务器
```

## 这就够了

PWA 的核心就这两个文件 + 三行注册代码。不需要框架，不需要复杂的配置。数独项目加起来不到 50 行，就实现了离线可用和安装到桌面的功能。

如果你想让 PWA 更完善，下一步可以：

- 用 Workbox 替代手写 Service Worker（Google 的库，帮你处理缓存策略）
- 添加推送通知（需要后端配合）
- 实现后台同步（Service Worker 的定期同步功能）
- 自动生成图标尺寸和 manifest（用 `vite-plugin-pwa`）

但对于一个简单的单页应用，手写的这 30 行已经足够了。
