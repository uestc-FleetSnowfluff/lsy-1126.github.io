# PaperMod → 幻梦 Illusion 迁移说明

迁移完成时间：2026-11-26
主题版本：[幻梦 Illusion v1.1.0](https://github.com/aizexintong/illusion)（GPLv3）
演示站：<https://illusion.azxt.org>

> 本次只改配置、布局与主题文件，**`content/posts/` 下的文章一个字都没动**。

---

## 0. 两个必须先知道的环境要求

### Dart Sass（不装会直接构建失败）

Hugo **v0.170 起移除了内置 LibSass**，幻梦的 `main.scss` 需要外部的 **Dart Sass** 才能编译。
而且必须是**官方独立二进制**——npm 上的 `sass` 是纯 JS 版，不支持 Hugo 需要的 `--embedded` 协议。

Windows 本地安装（任选其一）：

```powershell
# Chocolatey
choco install sass
# 或 Scoop
scoop install sass
# 或手动下载 https://github.com/sass/dart-sass/releases ，解压后把含 sass 的目录加进 PATH
```

验证：

```powershell
sass --version   # 能打印版本即可
hugo server
```

GitHub Actions 里已经在 `.github/workflows/hugo.yaml` 加了 `Install Dart Sass` 步骤，不需要你手动处理。

### 主题是「不带 content 的净化版」

幻梦官方仓库根目录自带的 `content/`（about/skills/tools/privacy/terms 演示页）在 Hugo v0.170 下**会被一并渲染**，
把 azxt 的演示「关于/隐私政策」发到你的域名下。所以 `themes/illusion/` 里刻意**没有** `content/` 目录，
只保留 `layouts/ assets/ static/ i18n/`。请**不要**把上游的 `content/` 拷回来。

---

## 1. 主题安装方式

`themes/illusion/` 是**直接随仓库提交的普通目录**（不是 git submodule），好处是：

- CI 不需要 `submodules: recursive`，也不怕上游仓库改名/删库；
- 想改主题 SCSS/TS 可以直接改，改完就在你自己的历史里。

代价是升级要手动。升级方法（可选）：

```powershell
git clone --depth 1 https://github.com/aizexintong/illusion "$env:TEMP\illusion-new"
# 只同步这四类目录，跳过 content/
robocopy "$env:TEMP\illusion-new\layouts" "themes\illusion\layouts" /E
robocopy "$env:TEMP\illusion-new\assets"  "themes\illusion\assets"  /E
robocopy "$env:TEMP\illusion-new\static"  "themes\illusion\static"  /E
robocopy "$env:TEMP\illusion-new\i18n"    "themes\illusion\i18n"    /E
```

旧的 `themes/PaperMod` 子模块与 `.gitmodules` 已移除。

---

## 2. 配置结构

| 文件 | 作用 |
| --- | --- |
| `hugo.yaml` | 站点主配置：`theme: illusion`、功能开关、菜单、Markdown/KaTeX、输出格式 |
| `data/theme.yaml` | 主题的唯一数据源：首页 Hero、关于、技能、友链、404、全部视觉开关 |
| `themes/illusion/` | 主题本体（不含 content） |
| `layouts/partials/_layout/footer.html` | 站点级覆盖：主题页脚 + Live2D 挂载点 |
| `layouts/partials/live2d.html` | 原样保留的看板娘代码（只改了与浮动按钮的避让 CSS） |
| `archetypes/default.md` | `hugo new` 模板，已预留 `image:` 封面字段 |

### 旧配置的对应关系

| PaperMod | 幻梦 |
| --- | --- |
| `theme: PaperMod` | `theme: illusion` |
| `params.profileMode`（首页头像/副标题） | `data/theme.yaml` → `home.hero` + `assets.avatar` |
| `params.socialIcons` | `data/theme.yaml` → `social.links` |
| `menu.main` | `hugo.yaml` → `menus.main` |
| `params.ShowReadingTime` 等 | 主题内置，改 `data/theme.yaml` → `layout.*` |
| 搜索页 `/search/` | 主题自带全站搜索：右下角放大镜 / `Ctrl+K` |
| `assets/css/extended/custom.css` | 幻梦不读这个文件（已失效，可自行删除） |
| `layouts/partials/extend_head.html` | 已删除（PaperMod 专有钩子） |
| `layouts/partials/extend_footer.html` | 已删除（幻梦无此钩子，改用 footer 覆盖） |
| `layouts/partials/math.html`（MathJax） | 已删除（幻梦原生 KaTeX） |

### 社交图标

`data/theme.yaml` → `social.links`，`show: true` 才显示：

```yaml
social:
  links:
    - platform: "github"
      url: "https://github.com/uestc-fleetsnowfluff"
      icon: "fab fa-github"      # Font Awesome 类名
      label: "GitHub"
      show: true
```

GitHub / 邮箱 / RSS 已按原配置开启，Twitter / Telegram / 微信保留为 `show: false`。

### 菜单

`hugo.yaml` → `menus.main`：首页 / 文章 / 归档 / 标签 / 友链。
原来的「搜索」菜单项去掉了——幻梦的搜索是模态框，没有独立页面，留着会是 404。
（`content/search.md` 已删除，`layout: search` 模板在幻梦里不存在。）

### 友链

友链数据在 `data/theme.yaml` → `friends.friends`，**不再写在 `content/links.md` 的表格里**
（幻梦的 `links` 模板只渲染数据，不渲染正文）。
`content/links.md` 现在只剩 Front Matter 的 `layout: "links"`。
4 位友链已迁移，头像占位图放在 `static/images/friends/`。

### 图片放哪里：`static/` 才会被发布，`assets/` 不会

`assets/` 只是资源管道的输入目录（`resources.Get`、图片处理），**里面的文件不会自动出现在网站根目录**；
只有 `static/` 下的文件会被原样发布到 `public/`。

踩过的坑：`data/theme.yaml` 里 `/images/avatar.png` 被引用了 4 处——首页「关于」卡片、友链「本站」、
OpenGraph 分享图、Schema.org logo——但文件当时只存在于 `assets/images/avatar.png`，
所以这 4 个位置全部 404，头像显示不出来。

现在的约定：

| 路径 | 说明 |
| --- | --- |
| `assets/images/avatar.png` | 1536×1536 母版，保留不动 |
| `static/images/avatar.png` | 实际发布的 512×512 版本（331 KB），页面与 OG 都引用它 |

**换头像**：替换 `static/images/avatar.png` 即可（想同时更新母版就两张一起换）。
用 512px 是因为主题里头像最大显示 140×140，2~3 倍屏也够；直接放 1536 的母版会白多下载 2 MB。

---

## 3. 文章封面（已预留）

幻梦原生就用 **`image`** 这个 Front Matter 字段（不是 `cover`），
卡片、文章页头图、OG 分享图、上下篇缩略图四处都读它：

```yaml
---
title: "文章标题"
date: 2026-11-26T10:00:00+08:00
image: "/images/covers/你的图片名.jpg"
---
```

- 图片放 `static/images/covers/`，路径写 `/images/covers/xxx.jpg`。
- **没写 `image` 时不会报错**：卡片自动退回毛玻璃/纯色占位，只是不显示图片区块。
  已验证 3 篇文章（都没有 `image`）构建正常、卡片无空 `<img>`。

---

## 4. 数学公式（改用原生 KaTeX）

- 站点配置 `params.enableMath: true` + `markup.goldmark.extensions.passthrough`（**两者缺一不可**）。
  passthrough 让 Goldmark 原样保留 `$...$` / `$$...$$` / `\(...\)` / `\[...\]`，公式里的 `_ * \` 不会被 Markdown 吃掉。
- 资源按需加载：只有正文里真的出现公式定界符的页面才请求 KaTeX（jsDelivr 主源 + unpkg 备用）。
- 单篇关闭：Front Matter 写 `math: false`。
- 原来的 MathJax partial 已删除。注意 MathJax 和 KaTeX 的语法覆盖不完全一致，
  复杂 `\begin{align*}` 之类如果渲染不理想，把该段公式改成 `$$ ... $$` 包裹的标准写法即可。

---

## 5. Live2D 看板娘

幻梦**没有** `extend_footer.html` 这种钩子（那是 PaperMod 的约定）。可用的挂载点只有两个：
覆盖 `layouts/_default/baseof.html`，或覆盖 `layouts/partials/_layout/footer.html`。
选择后者，改动最小：

```
layouts/partials/_layout/footer.html   # 主题同名文件的副本
                                       # 末尾多一行 {{ partial "live2d.html" . }}
```

⚠️ 这个文件是**整份复制**的。以后升级主题时，如果主题的 footer 有改动，需要手动同步一次。

### DOM 冲突（已处理，未删任何代码）

幻梦右侧有一组固定浮动按钮 `.scroll-buttons`（回顶 / 搜索 / 主题切换 / 到底部），
`position: fixed; right: var(--sp-2); bottom: var(--sp-2); z-index: 999`，
和看板娘（`right: 18px; bottom: 18px; z-index: 60`）**完全重叠**，会挡住画布点击。

处理方式：只加 CSS，把浮动按钮组整体左移到看板娘左侧并排（宽度按看板娘实际尺寸计算）：

```css
@media (min-width: 641px) {
  .scroll-buttons { right: calc(18px + 250px + 18px); }
}
@media (max-width: 640px) {
  .scroll-buttons { right: calc(10px + min(180px, 42vw) + 12px); bottom: 12px; }
}
```

原先针对 PaperMod `.top-link` 的两条规则已删除。JS 逻辑、分区点击、气泡、模型加载全部原样保留。
模型路径仍然是 `static/live2d/小爱弥斯_vts/小爱弥斯.model3.json`，构建产物已确认存在。

---

## 6. 站点定制（去掉主题的演示作者信息）

幻梦把作者信息写死在了三个地方，已全部处理：

| 位置 | 原文 | 来源 | 处理 |
| --- | --- | --- | --- |
| 页头副标题 | 爱则心痛 · azxt | 主题 `i18n` 的 `header_subtitle` | 覆盖 `layouts/partials/_layout/header.html`，直接删掉副标题那一行 |
| 页脚「关于我」 | …专注于**前端开发**、移动应用和新兴技术。 | 主题 `i18n` 的 `footer_about_desc` | 站点 `i18n/zh-CN.yaml` 覆盖为「ML 开发」 |
| 页脚「关于我」头像 alt | 爱则心痛 | 主题 `i18n` 的 `siteauthor` | 站点 `i18n/zh-CN.yaml` 覆盖为 `uestc-fleetsnowfluff` |
| 页脚最底部 | · **作者：爱则心痛** | 主题 `assets/ts/main.ts` 里**硬编码** | 改 TS，删掉作者署名段 |

要点：

- `i18n/zh-CN.yaml` 是**局部覆盖**，只写改动的 key，其余 200+ 条自动沿用主题。Hugo 的 i18n 是按 key 合并、站点优先。
- 「作者：爱则心痛」不是 i18n 能改掉的——`main.ts` 里写死了 `${authorLabel}<a href="https://github.com/aizexintong">爱则心痛</a>`，只能改源码。
  **升级主题时这一处会被覆盖**，需要重新删一次（搜 `footer_powered_author` / `爱则心痛`）。
- `header.html` 同样是整份覆盖的副本，升级时要和主题对比同步。
- 改完 TS 必须重新构建（`js.Build` 会重新编译 `main.js`）；`hugo server` 会自动重建。

现在页脚底部只剩：`由 Hugo 强力驱动 · 主题：幻梦 Illusion`。

---

## 7. 随机背景

`static/images/` 下放了 `bg1.jpg`~`bg4.jpg`，**每次加载页面随机展示一张**（刷新就换）。

配置在 `data/theme.yaml`：

```yaml
assets:
  background:
    random:
      enabled: true          # false 则退回 adaptive.url 单图
      images:
        - "/images/bg1.jpg"
        - "/images/bg2.jpg"
        - "/images/bg3.jpg"
        - "/images/bg4.jpg"
    adaptive:
      url: "/images/bg1.jpg" # 兜底图
```

实现方式（改了主题模板 `themes/illusion/layouts/_default/baseof.html`）：

- 服务端渲染时 `src` 先给列表第一张（禁用 JS 也能看到背景），
- 紧跟在 `<img>` 之后的内联脚本在**首次绘制前**随机替换 `src`，所以不会先闪一张再换。
- 模板逻辑：读 `assets.background.random`，未配置时退回原来的 `adaptive.url` 单图。

**加图/换图**：把文件丢进 `static/images/`，在 `random.images` 里加一行即可，不用改模板。
注意 `bg2.jpg` 有 2.3 MB，移动端首屏会偏慢；想更快可以压缩，或把它换成小图。

### 7.1 浅色模式下调暗背景

主题浅色模式是「背景图 + `rgba(253,248,245,0.6)` 的白色遮罩」。四张图本身偏亮（平均亮度 125~208），
合成后亮度被推到 **202~235**，所以整页看起来几乎是一片白。

现在在 `assets/css/custom.css` 里压暗（深色模式不受影响）：

```css
:root:not([data-theme='dark']) .bg-overlay-layer {
  background: rgba(253, 248, 245, 0.28);   /* 白遮罩 0.6 → 0.28 */
}
:root:not([data-theme='dark']) .bg-image {
  filter: brightness(0.72) saturate(1.05); /* 图本身压暗 */
}
```

合成后亮度约 **136~179**，图能看清了，整体仍然是浅色。

**调参**（都在 `assets/css/custom.css` 顶部两处）：

- `brightness(0.72)`：越小越暗。`0.60` 很暗 / `0.72` 当前 / `0.85` 接近原图
- 遮罩 alpha `0.28`：越小越清晰、整体越暗（改这个观感变化最直接）

**自定义样式入口**（新增，以后所有样式微调都写这里，不用碰主题 6500 行的 `main.scss`）：

- `layouts/partials/_layout/head/css.html`：主题同名文件的副本，只是多引了一个 `assets/css/custom.css`
- `assets/css/custom.css`：站点自定义样式，在主题样式之后加载，可直接覆盖

**踩坑记录**：只覆盖了 `.bg-overlay-layer` 这个元素，**没有**去改 `--overlay` 变量——那个变量同时被
`.mobile-menu-overlay`（移动端菜单遮罩）复用，直接改变量会把菜单背景也变透明。

---

## 8. 站点图标（favicon）

浏览器标签页的图标原来用的是主题自带的 `/favicon.svg`（幻梦 logo），现在换成头像 `assets/images/avatar.png`。

由头像生成三个文件，放在 `static/` 根目录：

| 文件 | 尺寸 | 用途 |
| --- | --- | --- |
| `favicon.png` | 64×64 | 主图标（Chrome / Edge / Firefox） |
| `favicon.ico` | 16 / 32 / 48 / 64 多尺寸 | 旧浏览器、Windows 快捷方式兜底 |
| `apple-touch-icon.png` | 180×180 | iOS 添加到主屏 |

图标声明改在主题 `themes/illusion/layouts/partials/_layout/head.html`：

```html
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="icon" type="image/png" sizes="64x64" href="/favicon.png">
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
```

注意：

- 主题的 `themes/illusion/static/favicon.svg` 仍会被发布，但已经没有引用，可以不管。
- **升级主题会覆盖 `head.html`**，这一处要重做（搜 `favicon.svg`）。
- 换图标：直接替换 `static/` 下这三个文件即可，尺寸保持一致。
- 浏览器对 favicon 缓存很顽固：本地看不到就用 `Ctrl+F5` 或开新标签页；线上还要等 GitHub Actions 部署完成。

---

## 9. 深色模式 FOUC（首屏闪白）修复

**现象**：深色模式用户打开页面，先看到一帧浅色（白屏），随后才切成深色。

**原因链**：

1. `data-theme` 由 `assets/ts/main.ts` 里的 `ThemeManager` 设置（`setThemeImmediately()` → `applyTheme()`）；
2. 这个 TS 被打包成 `main.js`，由 `layouts/partials/_layout/head/js.html` 用 **`<script defer src=...>`** 加载；
3. `defer` 必须等 HTML 解析完才执行，此时浏览器已经用 `:root` 的默认**浅色**变量渲染过一帧；
4. 而 `main.scss` 里**没有任何 `@media (prefers-color-scheme)` 兜底**（全站只有 `:root` 与 `[data-theme='light'|'dark']`），所以这段空窗期必然是浅色。

**修复**：在 `themes/illusion/layouts/partials/_layout/head.html` 的 `<head>` 顶部（`<meta charset>` / viewport 之后、**所有 CSS 之前**）插入一段**内联同步**脚本，把 `ThemeManager` 的决策逻辑提前：

```html
<script>
  (function () {
    var mode = 'system';
    try { mode = localStorage.getItem('theme') || 'system'; } catch (e) {}
    if (mode !== 'light' && mode !== 'dark') {
      mode = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }
    document.documentElement.setAttribute('data-theme', mode);
  })();
</script>
```

要点：

- **没有 `defer` / `async`**，同步执行；CSS 到达前 `data-theme` 已就位，首帧就是正确主题。
- 必须与 `ThemeManager` 的契约一致：localStorage 键名 `theme`，取值 `light` / `dark` / `system`，`system` 按 `prefers-color-scheme` 解析。**改任何一处都要同步另一处**。
- 脚本**只读不写** localStorage；运行时切换仍由 `ThemeManager` 负责，它随后会再赋一次同样的值（无副作用）。
- `localStorage` 在隐私模式/禁用存储时会抛异常，已用 `try/catch` 兜底为跟随系统；脏数据（非 light/dark）同样按系统偏好解析。
- `<meta charset>` 仍保持在最前（规范要求前 1024 字节内出现字符集声明），脚本紧随其后；脚本约 400 字节，不影响字符集探测。
- 位置验证（构建产物字符索引）：主题脚本 **281** → 首个 `<link rel="stylesheet">` **1160** → `fonts.css` 1190 → `main.js` 3049。
- 9 种组合（dark / light / system / 空值 / 脏数据 / 存储被禁用 × 系统深色/浅色）已用 Node 按真实产物里的脚本模拟执行，结果与 `ThemeManager` 完全一致。

**升级注意**：这段写在主题 `head.html` 里，升级主题会被覆盖，需要重做（搜 `prefers-color-scheme`）。

---

## 10. 备份 / 回滚

迁移前的状态都在 git 里（上一个提交）。

```powershell
git diff                # 看本次改动
git checkout -- .       # 丢弃全部未提交改动（会连带丢掉你之前的 custom.css 修改）
```

`assets/css/extended/custom.css` 是 PaperMod 专用样式，幻梦不会加载它。
它保留原样（含你未提交的修改），方便你对照旧样式，确认不需要后可以删掉。
