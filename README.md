# 在线视频解析安全中转页

这是一个纯静态 `player.html`，用于在 Tampermonkey 脚本和第三方解析站之间增加安全中转层。

## 目录结构

```text
safe-video-player/
├─ player.html
├─ README.md
├─ vercel.json
├─ netlify.toml
└─ assets/
   └─ .gitkeep
```

## 使用方式

部署后用 `url` 参数传入完整解析地址：

```text
https://mydomain.com/player.html?url=https%3A%2F%2Fjx.hls.one%2F%3Furl%3Dxxxxx
```

原始地址：

```text
https://mydomain.com/player.html?url=https://jx.hls.one/?url=xxxxx
```

更推荐在脚本中使用 `encodeURIComponent(parseUrl)`，避免嵌套参数被浏览器截断。

## 修改白名单

打开 `player.html`，同时修改两个位置：

1. `<meta http-equiv="Content-Security-Policy">` 里的 `frame-src` 和 `child-src`
2. JS 里的 `ALLOWED_PARSE_ORIGINS`

示例：

```js
const ALLOWED_PARSE_ORIGINS = new Set([
  "https://jx.hls.one",
  "https://www.jx.hls.one"
]);
```

只添加你信任的解析站 origin，不要把真实视频站加入这里。

## 安全限制

当前 iframe 使用：

```html
sandbox="allow-scripts allow-forms allow-presentation"
```

没有开启这些危险权限：

```text
allow-popups
allow-popups-to-escape-sandbox
allow-top-navigation
allow-top-navigation-by-user-activation
allow-downloads
```

因此可以限制第三方解析页：

- 弹出新窗口
- `target="_blank"` 打开窗口
- `window.open`
- 顶层页面跳转
- 自动下载
- 大部分外部 APP 协议唤醒

如果某个解析站因为严格 sandbox 无法播放，可以临时加 `compat=1` 开启兼容模式：

```text
https://mydomain.com/player.html?compat=1&url=https%3A%2F%2Fjx.hls.one%2F%3Furl%3Dxxxxx
```

兼容模式会把 sandbox 调整为：

```html
sandbox="allow-scripts allow-same-origin allow-forms allow-presentation"
```

`allow-same-origin` 可以改善解析站接口请求、播放器存储和移动端播放兼容性，但安全边界会弱于默认严格模式。建议只对确实需要的白名单解析站使用。

注意：纯前端无法修改第三方 iframe 内部代码，也无法彻底清除 iframe 内部页面自身渲染的广告元素。这个页面的作用是把第三方页面关进受限 iframe，阻止弹窗、跳转和越界行为。

## Tampermonkey 修改

在脚本里找到解析按钮点击事件附近的：

```js
window.open(parseUrl)
```

替换为：

```js
window.open("https://mydomain.com/player.html?url=" + encodeURIComponent(parseUrl))
```

如果你的脚本写的是：

```js
window.open(parseUrl, "_blank")
```

替换为：

```js
window.open("https://mydomain.com/player.html?url=" + encodeURIComponent(parseUrl), "_blank")
```

修改位置通常在这些关键词附近：

```text
解析
parseUrl
button.onclick
addEventListener("click"
GM_registerMenuCommand
```

## Vercel 部署

1. 把整个 `safe-video-player` 文件夹上传到 GitHub 仓库。
2. 在 Vercel 新建项目并选择该仓库。
3. Framework Preset 选择 `Other`。
4. Build Command 留空。
5. Output Directory 留空或填 `.`。
6. 部署完成后访问：

```text
https://你的域名/player.html
```

## Netlify 部署

1. 把整个 `safe-video-player` 文件夹上传到 GitHub 仓库。
2. 在 Netlify 选择 `Add new site`。
3. Build command 留空。
4. Publish directory 填 `.`。
5. 部署完成后访问：

```text
https://你的域名/player.html
```

## GitHub Pages 部署

1. 上传整个目录到 GitHub 仓库。
2. 进入仓库 `Settings`。
3. 打开 `Pages`。
4. Source 选择 `Deploy from a branch`。
5. Branch 选择 `main`，目录选择 `/root`。
6. 保存后访问 GitHub Pages 给出的地址。
