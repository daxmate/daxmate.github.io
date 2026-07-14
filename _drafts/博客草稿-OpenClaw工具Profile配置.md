# OpenClaw 工具 Profile 配置排查记

## 前言

最近在折腾 OpenClaw 通道配置的时候，遇到了一个挺有意思的问题，排查过程有点曲折，记录一下供参考。

事情是这样的——我的 `openclaw-weixin` 和 `qqbot` 通道一直在报一个告警：

> Agent "main" is routed from channel "openclaw-weixin" and "qqbot", but the message tool is unavailable for that agent; explicit channel actions such as sendAttachment, upload-file, thread-reply, or reply can fail. Add "message" to the agent tool allowlist, add "group:messaging", or switch the agent to a profile that includes messaging tools.

简单来说就是：微信和 QQ 通道的 main agent 没有 `message` 工具权限，发不了消息。

## 问题根因

翻了一下配置文件，发现全局设了 `tools.profile: "coding"`。这个 profile 是 OpenClaw 内置的几种预设之一，它包含了文件操作、运行时、网络、会话、记忆、cron 这些工具组，但**不包含** `group:messaging`（也就是 `message` 工具）。

而像 qqbot 和 openclaw-weixin 这类聊天通道，agent 必须能够调用 `message` 工具来发送回复和附件。没有这个工具，通道就一直在报警。

## 排查过程

### Step 1：发现问题

一开始先确认了配置文件里的 `tools.profile` 确实是 `"coding"`。

### Step 2：错误尝试

我最初的想法是在 main agent 级别加一个 `tools.allow`，把 `group:messaging` 加进去，像这样：

```json
{
  "id": "main",
  "tools": {
    "allow": ["group:messaging"]
  }
}
```

结果发现事情没这么简单——**agent 级别设置 `tools.allow` 是限制性的**，不是追加。也就是说，一旦你在 agent 级别设了 `allow`，那么这个 agent 就只能使用 `allow` 列表里的工具，全局 profile 里那些全都不算了。

于是 main agent 直接废了，连基本的对话能力都没了。

### Step 3：方案A——全局补救

后来大象同学尝试了另一条路：不碰 profile，直接在全局 `tools` 下加了一行：

```json
"tools": {
    "allow": ["group:messaging"],
    "profile": "coding",
    ...
}
```

但 OpenClaw 的 doctor 依然在报同样的告警，说明这么做也没有真正解决问题。

### Step 4：最终方案

正确的做法其实很简单——**把全局 profile 去掉，只给需要的 agent 单独设 profile**。

```json5
// 全局 tools 部分 - 去掉 profile
"tools": {
    "agentToAgent": { ... },
    "sessions": { ... },
    "web": { ... }
}

// writer agent 单独保留 coding profile
{
    "id": "writer",
    "tools": { "profile": "coding" }
}
```

这样 main agent 就没有限制（等于 full profile，什么工具都能用），而 writer agent 继续保持 coding 的受限环境。

## OpenClaw 内置 Profile 一览

顺手整理一下 OpenClaw 内置的几个工具 profile：

| Profile | 包含内容 |
|---------|---------|
| `minimal` | 仅 `session_status` |
| `coding` | `group:fs`, `group:runtime`, `group:web`, `group:sessions`, `group:memory`, `cron`, 目标管理, `image` |
| `messaging` | `group:messaging`（即 `message` 工具）, `sessions_list`/`history`/`send`, `session_status` |
| `full` | 无限制（等同于不设） |

## 经验教训

1. **agent 级别的 `tools.allow` 是限制性的**，不是增量追加。设了之后 agent 就只能用 allow 列表里的工具，全局 profile 的东西全部失效。
2. **改配置前先想清楚后果**——改错了 agent 可能直接失能，连对话都回不了。
3. **`tools.profile` 是 local onboarding 默认添加的**，如果你没主动改过但发现有这个设置，可以检查一下 onboarding 流程。
4. **把限制收窄到单个 agent**，而不是全局铺开，能让 main agent 保持最大灵活性。

## 小结

OpenClaw 的工具权限模型是**分层叠减**的：全局 profile 设了之后所有 agent 都受影响，agent 级别的 `allow` 又会进一步收窄。所以正确的做法是：保持全局开放（不设 profile），在特定 agent（比如 writer）上单独加限制比较安全。

希望这篇排查记录对遇到类似问题的朋友有帮助。
