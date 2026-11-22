# NetTinker 博客 · 使用说明

> 站点：<https://www.netlabnote.top>
> 仓库：<https://github.com/xiaoliangadmin/hugo-site>

这套博客的技术栈是 **Hugo（静态生成）+ PaperMod（主题）+ GitHub（源码托管）+ Cloudflare Workers（构建与托管）**。没有服务器，没有数据库，写文章就是写 Markdown，`git push` 之后一分钟左右自动上线。

---

## 1. 关键信息速查

| 项目 | 值 |
|---|---|
| 线上地址 | `https://www.netlabnote.top/` |
| Git 仓库 | `xiaoliangadmin/hugo-site`（public） |
| 生产分支 | `main` |
| Cloudflare 账号 | 你自己的账号（不记录在此文档） |
| CF 项目 | Workers 项目（名称见 Cloudflare 后台） |
| 构建命令 | `git submodule update --init --recursive && hugo --gc --minify` |
| 部署命令 | `npx wrangler deploy` |
| 输出目录 | `public`（由 `wrangler.jsonc` 指定） |
| 构建变量 | `HUGO_VERSION = 0.170.1` |
| 评论 | giscus，落在仓库 Discussions 的 `Announcements` 分类 |
| 主题版本 | PaperMod v8（git submodule，目录 `themes/hugo-PaperMod`） |

---

## 2. 本地环境

```bash
hugo version    # 期望 v0.170.x 且带 extended
git --version
```

- Hugo **必须是 extended 版**，否则主题的 SCSS 编译会失败
- 主题是 submodule，第一次拉仓库要执行：

```bash
git submodule update --init --recursive
```

---

## 3. 目录结构

```
hugo-site/
├── hugo.toml                  # 全站配置：标题、菜单、社交、评论
├── wrangler.jsonc             # Cloudflare 静态资源配置
├── archetypes/default.md      # 新建文章的模板（front matter）
├── layouts/
│   └── _partials/
│       ├── comments.html        # 自定义 giscus 评论模板
│       ├── toc.html             # 段落导航：宽屏右侧固定目录
│       └── extend_footer.html   # 段落导航：滚动高亮脚本
├── assets/
│   └── css/
│       └── extended/
│           └── toc-sticky.css   # 段落导航样式（自动并入 stylesheet）
├── static/
│   ├── favicon.ico 等图标
│   ├── safari-pinned-tab.svg
│   └── images/
│       └── covers/            # 文章封面（1200×630）
├── content/
│   ├── posts/                 # 所有文章，一个 .md 一篇
│   ├── about.md               # 关于页
│   ├── search.md              # 搜索页（layout: search）
│   ├── archives.md            # 归档页（layout: archives）
│   └── categories/
│       ├── tech/_index.md     # 分类显示名：技术
│       └── life/_index.md     # 分类显示名：生活
└── themes/hugo-PaperMod/      # 主题（submodule，不要手改）
```

---

## 4. 发一篇新文章

### 4.1 创建文件

```bash
cd ~/Desktop/hugo/hugo-site
hugo new content posts/文章英文短名.md
```

文件会生成在 `content/posts/`，front matter 取自 `archetypes/default.md`。

> **文件名用英文短横线**，它同时是 URL 的一部分，例如 `openwrt-ipv6-note.md` → `/posts/openwrt-ipv6-note/`。改名会改变链接，已发布文章尽量不要改。

### 4.2 填写 front matter

```toml
+++
title = "R5S 上把 IPTV 组播转单播"
date = 2026-11-20T22:10:00+08:00
draft = false
description = "一句话摘要，会出现在首页列表、搜索结果和分享卡片里"
categories = "tech"
tags = ["OpenWrt", "R5S", "IPTV"]
cover = { image = "/images/covers/iptv.jpg", alt = "封面说明", relative = false }
+++
```

| 字段 | 说明 |
|---|---|
| `title` | 标题，不要带结尾标点 |
| `date` | **必须带时区** `+08:00`。Hugo 默认**不发布未来时间的文章**，日期写成今天晚些时候也会导致不上线 |
| `draft` | `true` 只在本地预览可见；`false` 才会发布 |
| `description` | 必填，直接影响首页摘要和 SEO |
| `categories` | 目前只有 `tech` / `life`，对应「技术」「生活」 |
| `tags` | 自由填写，建议 2–5 个，用来做细分检索 |
| `cover` | 封面。图片放 `static/images/covers/`，这里写 `/images/covers/xxx.jpg`，并保持 `relative = false` |

### 4.3 写正文

就是普通 Markdown：

```markdown
## 二级标题

正文段落。

- 列表项
- 列表项

​```bash
命令
​```
```

正文里插图：把图片放到 `static/images/<文章名>/`，然后：

```markdown
![图片说明](/images/xxx/a.png)
```

### 4.4 本地预览

```bash
hugo server -D
# 打开 http://localhost:1313
```

`-D` 表示连草稿一起渲染。改文件会自动刷新。

### 4.5 发布上线

1. 把 `draft` 改成 `false`
2. 提交并推送：

```bash
git add -A
git commit -m "post: 标题"
git push origin main
```

Cloudflare 会自动构建，通常 30–60 秒完成。去 `https://www.netlabnote.top` 刷新即可看到。

---

## 5. 日常修改

### 5.1 站点基本信息

都在 `hugo.toml`：

```toml
title = "NetTinker · 子网掩码"
copyright = "© 2026 NetTinker"

[params]
  description = "网络折腾、软路由、自建服务与骑行生活"
  author = "NetTinker"

  [params.homeInfoParams]
    Title = "你好，我是 NetTinker"
    Content = "折腾家庭网络与软路由，也骑车、也读书。"
```

改完 push 即可。浏览器有缓存，必要时强刷（Cmd+Shift+R）。

### 5.2 导航菜单

`hugo.toml` 的 `[menu]` 段，`weight` 越小越靠前：

```toml
[[menu.main]]
  identifier = "about"
  name = "关于"
  url = "/about/"
  weight = 60
```

新增一个菜单项就复制一段，改 `identifier` / `name` / `url` / `weight`。

### 5.3 社交图标

```toml
[[params.socialIcons]]
  name = "github"          # 可选 github / youtube / rss / x / mail ...
  url = "https://github.com/xiaoliangadmin"
```

删掉整段就少一个图标。

### 5.4 关于页

直接改 `content/about.md`，正文随便写。

### 5.5 新增一个分类

1. 建目录说明文件：

```bash
mkdir -p content/categories/notes
cat > content/categories/notes/_index.md <<'EOF'
+++
title = "笔记"
+++
分类描述文字。
EOF
```

2. 文章 front matter 写 `categories = "notes"`，页面上就显示「笔记」。

> 分类的 URL 用英文（`/categories/notes/`），显示名由 `_index.md` 的 `title` 决定。

### 5.6 换封面或加封面

1. 图片放进 `static/images/covers/`，建议 1200×630，JPEG，控制在 100KB 以内
2. 文章 front matter 里改 `cover.image`

---

## 5.7 段落导航（右侧固定目录）

文章页在宽屏下会在正文右侧常驻一条目录，滚动时高亮当前所在段落。窄屏（<1280px）自动退回正文顶部的折叠目录。

涉及三个文件，都属于「站点覆盖主题」，升级 PaperMod 不会被覆盖：

| 文件 | 作用 |
|---|---|
| `layouts/_partials/toc.html` | 覆盖主题同名 partial。宽屏输出 `nav.toc-side`，窄屏保留原生 `details` |
| `assets/css/extended/toc-sticky.css` | 固定定位、层级缩进、当前段高亮样式（PaperMod 自动合并） |
| `layouts/_partials/extend_footer.html` | scrollspy 脚本，给当前段加 `.is-active` |

行为规则：

- 全站开关仍是 `hugo.toml` 里的 `ShowToc = true`；某篇文章写 `ShowToc = false` 就不出现目录（`about.md` 就是这种）
- 目录项由 Hugo 的 `.Fragments.Headings` 生成，**只跟随 Markdown 标题**（`##` / `###` / `####`），不会出现空项
- 宽屏显示断点是 `1280px`；侧栏位置按 `--main-width` 计算，自动贴着正文右边缘

常用微调：

```css
/* assets/css/extended/toc-sticky.css */
top: 88px;                              /* 顶部起始位置，避开导航栏 */
width: 196px;                           /* 目录宽度 */
@media screen and (min-width: 1280px)   /* 显示断点，屏幕窄可改小 */
```

```js
/* layouts/_partials/extend_footer.html */
var OFFSET = 96;   // 判定「当前段落」的顶线，调大 → 更早切换高亮
```

想临时关掉这一整套：把 `hugo.toml` 的 `ShowToc` 改回 `false`，或者删掉 `assets/css/extended/toc-sticky.css`（侧栏会失去定位样式）。

---

## 6. 评论管理

评论数据全部存在 GitHub Discussions，不在数据库里。

| 要做的事 | 在哪里做 |
|---|---|
| 查看 / 回复评论 | 仓库 → Discussions → `Announcements` 分类 |
| 删除评论 | 同上，GitHub 的删除操作 |
| 关闭单篇评论 | 文章 front matter 加 `comments = false` |
| 关闭全站评论 | `hugo.toml` 的 `[params]` 里 `comments = false` |
| 调整评论参数 | `hugo.toml` 的 `[params.giscus]` |

访客需要登录 GitHub 才能评论，这是 giscus 的机制。

---

## 7. 部署与排错

### 7.1 查看构建状态

Cloudflare 后台 → Workers 和 Pages → 本项目 → **部署** 标签页。
每条记录后面有「构建」链接，点进去看完整日志。

### 7.2 回滚

两种方式：

- **平台侧**：部署列表里找到上一个「就绪」的版本，用它的「推出」把流量切回去（快，几秒）
- **代码侧**：`git revert <坏提交>` 然后 push（推荐，保持历史一致）

### 7.3 重新构建

Cloudflare 后台部署详情页有「重试构建」或重新推送一次空提交：

```bash
git commit --allow-empty -m "chore: trigger rebuild" && git push
```

---

## 8. 主题与版本升级

**不要直接改 `themes/hugo-PaperMod/` 里的文件**，升级会覆盖。要改样式就放 `layouts/` 或 `assets/` 里做覆盖。

升级主题：

```bash
git submodule update --remote themes/hugo-PaperMod
git add themes/hugo-PaperMod && git commit -m "chore: 升级 PaperMod"
git push
```

注意：**主题对 Hugo 版本有下限要求**。升级后如果构建报 `hugo vX.Y or greater is required`，就去 Cloudflare 后台把 `HUGO_VERSION` 调高；反过来，Hugo 版本太新而主题没适配，就降 Hugo。

本地版本和线上版本要尽量保持一致，否则会出现「本地正常、线上报错」。

---

## 9. 备份

- 源码、文章、图片全部在 GitHub 仓库里，这就是主备份
- 主题在 submodule，属于外部仓库，不算备份项
- 图片不要只存本地：`static/images/` 已随仓库提交，等于有一份远端副本
- 建议每隔几个月 `git clone` 一份到移动硬盘或 NAS

---

## 10. 常见问题

| 现象 | 原因 | 处理 |
|---|---|---|
| 首页没有刚写的文章 | `draft = true`，或 `date` 是未来时间 | 改 `draft = false`；确认 date 不晚于当前时间 |
| 构建失败：`hugo vX or greater is required` | 线上 Hugo 版本低于主题要求 | CF 后台改 `HUGO_VERSION` |
| 构建失败：`module "PaperMod" not found` | 主题目录名与 `theme` 配置不一致 | 目录是 `hugo-PaperMod`，配置要写 `theme = "hugo-PaperMod"` |
| 线上样式全无、纯文本 | 主题没拉下来 | 构建命令必须带 `git submodule update --init --recursive` |
| 封面 404 | 路径写成相对路径 | 用 `/images/covers/xxx.jpg` 且 `relative = false` |
| 评论区空白 | 仓库变私有，或 Discussions 关闭 | 仓库保持 public，Settings 里确认 Discussions 已勾选 |
| 宽屏看不到右侧目录 | 窗口窄于 1280px，或该文 `ShowToc = false` | 拉宽窗口到 1280 以上；检查 front matter |
| 改了配置线上没变 | Cloudflare 边缘缓存 | 强刷，或在 CF 里 purge cache |
| 中文标点导致字数统计奇怪 | 没开 CJK 优化 | 确认 `hasCJKLanguage = true` |
| `hugo server` 报 SCSS 错误 | Hugo 不是 extended 版 | `brew install hugo` 重装 extended |

---

## 11. 命令速查

```bash
# 本地预览（含草稿）
hugo server -D

# 本地构建，检查是否有错
hugo --gc --minify

# 新建文章
hugo new content posts/xxx.md

# 发布
git add -A && git commit -m "post: xxx" && git push origin main

# 主题升级
git submodule update --remote themes/hugo-PaperMod

# 强制重建
git commit --allow-empty -m "chore: rebuild" && git push
```

---

## 12. 发布前检查清单

- [ ] `draft = false`
- [ ] `date` 不是未来时间，且带 `+08:00`
- [ ] `description` 写了
- [ ] `categories` 是 `tech` 或 `life`
- [ ] `tags` 2–5 个
- [ ] 封面路径正确，图片已放进 `static/images/covers/`
- [ ] 正文里的图片路径以 `/images/` 开头
- [ ] `hugo server -D` 本地看过一遍
