---
layout: single
title:  "网盘没有下载按钮的时候——直链下载助手的逆向分析笔记"
date:   2026-07-28 09:22:00 +0800
categories:
  - 编程笔记
tags:
  - 逆向
  - 用户脚本
  - React
  - 夸克网盘
---

如果你用过夸克网盘的网页版分享页面，可能会遇到一个困惑：页面上没有下载按钮。

只能「保存到网盘」或「去客户端查看」，下载入口被彻底藏起来了。这不是 Bug，是刻意的设计——文件夹和大文件分享页不提供直接下载，必须转存后才能在自己的主页下载。

社区里有一个叫「网盘直链下载助手」的用户脚本可以解决这个问题，但它不能直接在分享页面工作，需要先转存再去主页操作。这篇文章不是为了介绍怎么用，而是借这个脚本的夸克模块，聊聊前端逆向的三个核心技巧。

## 问题的本质

先说清楚为什么分享页没有下载入口。

夸克网盘的产品逻辑是这样的：单个小文件在分享页可能允许直接下载，但文件夹（尤其是一堆文件，比如 359 个）必须转存到用户自己的网盘。这么做一是为了强制注册和登录，二是减少分享页面的带宽消耗。

所以那个脚本的作者也做了对应处理——脚本的 `quark.getPCSLink` 函数里有这样一段逻辑：

```javascript
if (pt === 'home') {
  // 主页逻辑：发送请求拿直链
} else {
  // 分享页逻辑：直接拒绝
  return message.error('提示：请转存到自己网盘后去网盘主页下载！');
}
```

不是脚本做不到，是作者主动屏蔽了。理由不难理解：分享页的文件夹结构复杂，获取深层直链需要额外计算 `share_id` 和 `parent_id`，容易出问题。与其让用户在一个不稳定的路径上折腾，不如强迫先转存。

所以实际的使用流程是两步：先在分享页点击「保存到网盘」，然后回到网盘主页用脚本获取直链。但真正有意思的部分是脚本在主页到底做了什么。

## 核心技巧一：穿透 React 虚拟 DOM

传统爬虫的思路是解析 HTML。但现在的网站（夸克、阿里云盘）都是用 React 渲染的，DOM 结构变化频繁，靠解析 CSS 类名来拿数据很不稳定。

这个脚本用了一个更直接的方法——读内存。

```javascript
base.findReact = function(dom, traverseUp = 0) {
  const key = Object.keys(dom).find(key => {
    return key.startsWith("__reactFiber$") || key.startsWith("__reactInternalInstance$");
  });
  const domFiber = dom[key];
  if (domFiber == null) return null;

  const GetCompFiber = fiber => {
    let parentFiber = fiber.return;
    while (typeof parentFiber.type == "string") {
      parentFiber = parentFiber.return;
    }
    return parentFiber;
  };
  let compFiber = GetCompFiber(domFiber);
  for (let i = 0; i < traverseUp; i++) {
    compFiber = GetCompFiber(compFiber);
  }
  return compFiber.stateNode || compFiber;
};
```

这段代码做的事情是：React 在 DOM 元素上挂载了一个隐藏属性叫 `__reactFiber$`（不同版本可能叫 `__reactInternalInstance$`），通过这个属性可以拿到 Fiber 节点，再沿着 `fiber.return` 向上遍历，找到包含 `stateNode`（组件实例）的父级节点，最终把组件里的 `props` 和 `state` 掏出来。

拿到组件实例后，就很容易提取文件列表了：

```javascript
quark.getSelectedList = function() {
  let reactDom = document.getElementsByClassName('file-list')[0];
  let reactObj = base.findReact(reactDom);
  let props = reactObj.props;

  if (props) {
    let fileList = props.list || [];
    let selectedKeys = props.selectedRowKeys || [];
    // 用 fid 匹配，构建选中文件列表
    fileList.forEach((val) => {
      if (selectedKeys.includes(val.fid)) {
        selectedList.push(val);
      }
    });
  }
  return selectedList;
};
```

`props.list` 是当前页面的全部文件，`props.selectedRowKeys` 是用户勾选的文件 ID 数组。不需要解析 DOM，不需要模拟点击，直接从内存读取。

这种「React 数据劫持」的技巧在很多用户脚本和浏览器扩展中都有应用。它利用了 React 调试属性的存在——这些属性本身是为开发者工具设计的，但在用户脚本里，它们是最稳定的数据入口。比依赖 CSS 类名或者 DOM 结构要可靠得多，因为 React 内部 Fiber 架构的变动频率远低于 UI 层。

## 核心技巧二：直接调用内部 API

拿到文件 ID 后，脚本并没有去解析页面上的「下载」按钮或者模拟用户点击，而是直接向夸克的后端接口发了一个 POST 请求：

```javascript
let res = await base.post(pan.pcs[0], {
  "fids": fids
}, {"content-type": "application/json;charset=utf-8"});
```

`fids` 是文件 ID 数组，`pan.pcs[0]` 是夸克获取下载链接的 API 地址（由脚本的远程配置动态提供）。如果返回码 `code` 为 0，说明成功了，`res.data` 里就包含了每个文件的 `download_url`——真正的下载直链。

关键点在于：这个请求是在浏览器里发的。脚本运行在夸克网页的上下文中，`GM_xmlhttpRequest`（或直接 `fetch`/`XMLHttpRequest`）会携带用户的登录 Cookie，所以服务器认为这是一个合法的前端请求，返回了直链。

这种直接调用内部 API 的做法比模拟点击下载按钮要高效得多。它也揭示了一个事实：很多前端「限制」只是在 UI 层做的——后端接口是开放的，只要请求参数对、Cookie 对，就能拿到数据。页面上没有下载按钮，不代表后端没有提供下载链接。

## 核心技巧三：Cookie 透传

拿到直链后还有一个问题：夸克的直链校验很严格，它不仅要看链接本身，还要看请求携带的 Cookie。直接把直链复制到浏览器地址栏是打不开的。

脚本的解决方案是：在生成的下载命令里带上当前的 Cookie。

```javascript
quark.convertLinkToAria = function(link, filename, ua) {
  filename = filename.replace(' ', '_');
  return encodeURIComponent(
    `aria2c "${link}" --out "${filename}" --header "Cookie: ${document.cookie}"`
  );
};
```

这样生成的命令可以直接丢给 Aria2 或 IDM，下载器在请求直链时会带上正确的 Cookie，服务器就会放行。

这种做法其实是利用了「Cookie 是当前登录状态」这一事实。用户脚本在页面上下文中，可以拿到页面的 `document.cookie`，但外部下载器没有这个上下文，所以需要显式传递。这个技巧不仅适用于夸克，也适用于其他有 Cookie 校验的网盘和文件服务。

---

## 三条经验

回过头来看，这个脚本的核心逻辑并不复杂，但它展示了三个很重要的思路：

1. **UI 是表象，数据在内存里。** 现代前端框架把数据存在组件实例里，与其解析界面不如直接读内存。React 的 Fiber 架构调试属性是最稳定的数据入口。
2. **前端限制和后端限制是两回事。** 页面没有下载按钮不代表后端没有接口，有时候只需要模拟一个带正确 Cookie 的请求而已。
3. **Cookie 是你的身份证。** 在浏览器里能做的事，在外部工具里不一定能做。关键是要把当前上下文的认证信息传递出去。

如果想亲自验证，可以在脚本的 `quark.getPCSLink` 里加一句 `console.log(res)`，看看浏览器控制台打印出来的 `res.data` 长什么样——看到那个包含 `download_url` 的 JSON 结构，会比读这篇文章更直观。
