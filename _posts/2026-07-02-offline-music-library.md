---
layout: single
title:  "搭建一个属于自己的离线音乐库"
date:   2026-07-02 14:30:00 +0800
categories:
  - other
tags:
  - 音乐
  - 离线
  - iOS
  - Cosmos
  - CocoDownloader
  - MusicBrainz Picard
---

今天不知道为什么突然想听歌。

不是打开网易云点推荐的那种听。是想要一首一首地，把自己喜欢的歌找回来。

现在的音乐服务都是订阅制——交了月费，几万首歌随便听。听起来很方便，但用久了发现一个问题：推送的东西越来越多，自己主动去找的东西越来越少。算法确实能帮你发现新歌，但它也悄悄剥夺了另一种乐趣——偶然看到一张专辑封面、顺着一个名字摸过去、听了一个版本不满意换另一个版本——这些"慢"的过程。

今天花了几个小时，搭了一套完全属于自己的离线音乐体系。

---

## 播放器：Cosmos Music Player

iOS 上开源离线播放器的选择不多，最后选了 [Cosmos Music Player](https://github.com/PackageFoundation/Cosmos)。优点是可以直接从 iCloud 加载音乐文件，不需要 iTunes 同步那一套。

中文歌词支持一般，但中文歌有没有歌词问题不大——大部分都听过，不靠歌词也知道在唱什么。

---

## 下载：CocoDownloader

现在的歌都被大平台垄断了，想下载没那么容易。yt-dlp 能从 YouTube 下，但速度慢。国内的网站要么不好下，要么只能下 30 秒试听。

后来在 GitHub 上找到了一个叫 [CocoDownloader](https://github.com/markcxx/coco-downloader) 的开源项目。不是全部都能下，但日常够用。

最开始想用 pyinstaller 打包，折腾了一番也能跑起来，但遇到两个问题：桌面版的数据和网页版没有同步，功能不完全一致；而且 pyinstaller 出来的包体积太大，动辄几百 MB。

后来想起刚写完的 [Pake](https://github.com/tw93/Pake)——用 Rust 写的网页打包器，基于系统 webview，打出来的包通常不到 10MB。操作很简单：

```bash
brew install pake
```

先把 CocoDownloader 跑起来：

```bash
git clone https://github.com/markcxx/coco-downloader
cd coco-downloader
npm install
npm run dev
```

然后直接用 Pake 打包：

```bash
pake http://localhost:3000 --name CocoDownloader --icon desktop/app/resource/images/logo/CocoDownloader.ico
```

运行完当前目录就会生成一个 `CocoDownloader.dmg`，打开拖到 Applications 文件夹即可。体验出奇地好——它本质上就是个独立窗口的网页版，所有功能完整，UI 一致，歌词也能正常显示。

项目自带的 `.ico` 清晰度不高，我顺手用 SVG logo 生成了一套 macOS 原生图标：

```bash
# 先生成各个尺寸的 png
brew install librsvg
mkdir MyIcon.iconset

rsvg-convert -w 16 -h 16 input.svg -o MyIcon.iconset/icon_16x16.png
rsvg-convert -w 32 -h 32 input.svg -o MyIcon.iconset/icon_16x16@2x.png
rsvg-convert -w 32 -h 32 input.svg -o MyIcon.iconset/icon_32x32.png
rsvg-convert -w 64 -h 64 input.svg -o MyIcon.iconset/icon_32x32@2x.png
rsvg-convert -w 128 -h 128 input.svg -o MyIcon.iconset/icon_128x128.png
rsvg-convert -w 256 -h 256 input.svg -o MyIcon.iconset/icon_128x128@2x.png
rsvg-convert -w 256 -h 256 input.svg -o MyIcon.iconset/icon_256x256.png
rsvg-convert -w 512 -h 512 input.svg -o MyIcon.iconset/icon_256x256@2x.png
rsvg-convert -w 512 -h 512 input.svg -o MyIcon.iconset/icon_512x512.png
rsvg-convert -w 1024 -h 1024 input.svg -o MyIcon.iconset/icon_512x512@2x.png

# 合成 icns 图标
iconutil -c icns MyIcon.iconset
```

把生成的 `MyIcon.icns` 覆盖到 App 的图标，`Get Info` 那边就好看多了。顺手在项目的 issue 里分享了操作方法，也算回馈了社区。

### 让 App 持久运行

用了一阵发现一个问题：Pake 本质上是把网页包成独立窗口，如果本地服务停了，App 也就黑了。上面 `npm run dev` 的方式，终端一关就没了。

解决方法是把 CocoDownloader 的 Next.js 服务作为后台守护进程运行。

先把 `next.config.ts` 中的 `output: standalone` 注释掉，然后编译：

```bash
npm run build
```

然后在 `~/Library/LaunchAgents/` 下新建一个 plist 文件，让系统帮忙管理进程生命周期：

```bash
# 创建日志目录
mkdir -p ~/Library/Logs/CocoDownloader
```

创建 `~/Library/LaunchAgents/com.coco.downloader.plist`，填写以下内容（替换占位符）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.coco.downloader</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/node</string>
        <string>项目的绝对路径/node_modules/.bin/next</string>
        <string>start</string>
    </array>
    <key>WorkingDirectory</key>
    <string>项目的绝对路径</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/你的用户名/Library/Logs/CocoDownloader/coco-downloader.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/你的用户名/Library/Logs/CocoDownloader/coco-downloader.err</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PORT</key>
        <string>3000</string>
    </dict>
</dict>
</plist>
```

加载守护进程：

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.coco.downloader.plist
```

之后这个服务就会开机自启、崩溃自动重启。Pake 打包的 App 随时打开都能用，再也不怕终端关了。

当然，如果自己有服务器，也可以直接把项目部署到服务器上，本地 App 指向远程地址——那就更彻底了。

---

## 标签：MusicBrainz Picard

下载下来的歌有些缺少歌曲信息——歌手名空着、专辑名空着、封面没有。找到了 [MusicBrainz Picard](https:
//picard.musicbrainz.org/)，可以自动搜索歌曲的元信息并写入文件。把拖进来的歌匹配一下，点保存，所有的标签就补好了。

---

## 整个流程

```
CocoDownloader（pake版）→ MusicBrainz Picard → iCloud → Cosmos
```

从找歌、下载、补标签、同步到手机、打开播放——每一步都在自己掌控之下。

---

想听歌的时候就下载一点，慢慢地，播放列表里就全是"自己的歌"了。
