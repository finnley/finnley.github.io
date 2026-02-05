+++
title = '主题hugo-theme-drone使用'
date = 2026-01-04T14:22:41+08:00
draft = true
categories = [ "Hugo" ]
tags = [ "hugo", "drone" ]
+++

说实话，让非技术背景或者只是想写博客的用户去配置 Node.js、PostCSS 和 Go Modules，确实是**非常糟糕的用户体验**。

为了达到**“下载即用”**（无需 `npm install`，无需配置 Node 环境）的效果，最好的方案是将**复杂性留给开发者（我自己），便利性留给用户**。

这里有两个更简单、更实用的方案：

---

## 方案一：提交“编译后的成品” (最推荐，真正的零配置)

这个方案的核心思想是：**我在本地开发时把 CSS 编译好，然后把生成的 CSS 文件也提交到 GitHub。** 这样用户下载主题时，拿到的是已经处理好的 CSS，Hugo 就不需要再去调用 PostCSS 了。

### 1. 修改 `head.html` 逻辑

我需要让 Hugo 优先使用现成的 CSS，而不是每次都去重新编译。

修改我的 `layouts/partials/head.html`：

```html
{{/* 尝试获取已经编译好的资源（如果存在） */}}
{{ $styles := resources.Get "css/compiled.css" }}

{{/* 如果没有编译好的资源（说明是开发模式），则进行 PostCSS 处理 */}}
{{ if not $styles }}
  {{ $styles = resources.Get "css/main.css" | css.PostCSS }}
  {{/* 在开发时，可以将处理后的资源复制到 assets/css/compiled.css (可选优化) */}}
{{ end }}

{{/* 无论哪种方式，最终输出链接 */}}
{{ if hugo.IsProduction }}
  {{ $styles = $styles | minify | fingerprint }}
{{ end }}

<link rel="stylesheet" href="{{ $styles.RelPermalink }}">

```

### 2. 开发者的操作流程 (作为主题作者的我)

每次我修改了 CSS 或 HTML 样式后，我需要手动运行一次命令生成最终的 CSS 文件，并将其提交。

我可以写一个简单的 npm script (在主题的 `package.json` 中)：

```json
"scripts": {
  "build:css": "tailwindcss -i ./assets/css/main.css -o ./assets/css/compiled.css --minify"
}

```

**发布主题前的步骤：**

1. 我在本地运行 `npm run build:css`。
2. 这会在 `assets/css/` 下生成一个纯净的 `compiled.css`。
3. 我把这个 `compiled.css` 连同其他文件一起 `git commit` 并推送到 GitHub。

### 3. 用户的体验

* **用户操作**：`git clone` 主题 -> `hugo server`。
* **结果**：直接运行成功。不需要 `npm install`，不需要 `node_modules`。
* **代价**：如果用户想修改我的主题样式（比如改颜色），他们自己需要安装 Node 环境。但如果只是写文章，完全不需要。

---

## 方案二：利用 Hugo 的 `resources` 缓存机制 (进阶)

Hugo 有一个特性：如果我把 `resources/_gen` 目录提交到 Git，Hugo 会直接使用里面的缓存文件，而跳过构建步骤。

### 1. 调整 `.gitignore`

通常我们会忽略 `resources` 目录，但为了让用户免配置，我需要**取消忽略**它。

在主题的 `.gitignore` 中，**移除** 或 **注释掉**：

```bash
# resources/  <-- 移除这一行，允许提交 resources 目录

```

### 2. 开发者的操作流程

1. 在我的开发机上，运行一次 `hugo` (构建命令)。
2. Hugo 会在主题根目录下生成 `resources/_gen/images/...` 和 `resources/_gen/assets/...`。
3. 将这些生成的文件全部提交到 Git：
```bash
git add resources
git commit -m "chore: commit generated assets for zero-config usage"

```



### 3. 用户的体验

* **用户操作**：下载主题 -> `hugo server`。
* **原理**：Hugo 启动时发现 `resources` 目录下已经有指纹匹配的文件了，它就会直接用，不再呼叫 PostCSS。
* **优点**：不需要改代码逻辑，只需要改 Git 策略。

---

## 方案三：使用 GitHub Actions 自动发布 CSS (适合 CI/CD)

如果我不想把生成的 CSS 放在源码里，我可以利用 GitHub Actions。

1. 用户 Fork 我的仓库。
2. 配置一个 GitHub Action，当有代码更新时，自动运行 `npm install` 和 `hugo` 构建。
3. 将构建好的 `public` 文件夹部署到 GitHub Pages。

**虽然这看起来很自动化，但对于想在本地 `hugo server` 预览的用户来说，依然解决不了问题。** 所以对于主题开发，**方案一**或**方案二**是最好的。

### 总结建议

**强烈建议我使用“方案一”的变体：直接生成静态 CSS。**

1. 在主题里直接用 Tailwind CLI 命令把 CSS 编译成一个标准的 `static/css/style.css` 文件。
2. 在 `head.html` 里直接引用这个静态文件。
3. 开发时我自己开着 Tailwind 的监听模式 (`--watch`)。
4. 发布时，把这个 `style.css` 提交上去。

这样我的主题就变成了一个**普通的、没有任何 Node 依赖的 Hugo 主题**，任何小白用户下载就能用，这是最友好的方式。