## 坑点

### tina cms

* token、id 改名
* github pages 部署存在故障


### npm 依赖报错：


1. 删除 node_modules、package-lock.json、`npm cache clean --force && npm install`
2. [withastro/astro#6830](https://github.com/withastro/astro/issues/6830)

### 时区问题

npm dev 文章正常显示，但 build + http server 文章却没有按预期显示的原因：当 dev 状态的时候，不检查后面，如果是 build 状态就要检查发布时间了。

时间格式参考：

- 2025-11-24 23:13:43+08:00
- 2025-03-20T03:15:57.792Z

注意检查主题有关于配置时间、时区的配置文件。

### action

根目录创建 `.nojekyll` 文件，并 `.github\workflows\deploy.yml`


```yml
name: GitHub Pages Astro CI

on:
  # 每次推送到 `main` 分支时触发这个“工作流程”
  push:
    branches: [ main ]
  # 允许你在 GitHub 上的 Actions 标签中手动触发此“工作流程”
  workflow_dispatch:
  
# 允许 job 克隆 repo 并创建一个 page deployment
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your repository using git
        uses: actions/checkout@v4
      - name: Install, build, and upload your site
        uses: withastro/action@v5  # 更新为最新的 withastro action 版本

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4  # 已是 v4 版本，确保版本匹配
```

pnpm更新：`pnpm install --frozen-lockfile`

<img width="1157" height="539" alt="image" src="https://github.com/user-attachments/assets/c356fccb-d3e7-4aa1-9c1e-4c192e1694f1" />



## 简要配置定位

* `\src\config.ts` 博客资料页修改
* `\src\pages\index.astro` 欢迎页修改
* `\src\constants.ts` 社交链接修改
* `src\pages\about.md` 关于
* `src\components\Header.astro` 头部
* `\src\components\Footer.astro` 页脚信息

图标网站参考：

* https://icon-sets.iconify.design
* https://icones.js.org


## 固定URL，文章移动到任意文件夹保持固定链接

URL效果： `localhost:4321/posts/{slug}`

使用方式，全文替换如下文件

```astro
\src\pages\posts\[...slug]\index.astro
---
import { getCollection } from "astro:content";
import PostDetails from "@/layouts/PostDetails.astro";
import { getPath } from "@/utils/getPath";
import getSortedPosts from "@/utils/getSortedPosts";
export async function getStaticPaths() {
const posts = await getCollection("blog", ({ data }) => !data.draft);  // 获取所有非草稿文章
const postResult = posts.map(post => ({
params: { slug: getPath(post.id, post.filePath, false) },  // 使用 getPath 生成路径
props: { post },
}));
return postResult;
}
const { post } = Astro.props;
const posts = await getCollection("blog");
const sortedPosts = getSortedPosts(posts);
// 渲染 PostDetails 组件，显示当前文章和排序后的所有文章
<PostDetails post={post} posts={sortedPosts} />
```


```ts
\src\utils\getPath.ts
import { slugifyStr } from "./slugify";  // 引入 slugify 工具
// 生成文章的 URL 路径
export function getPath(id: string, filePath: string | undefined, includeBase = true) {
// 使用 slugifyStr 生成符合 URL 格式的 slug
const slug = slugifyStr(id);  // 使用 id 生成 slug，确保符合 URL 格式
console.log(filePath);  // 这样使用后，警告就会消失
// 仅返回 slug，生成固定路径
return includeBase ? </span><span class="token string">/posts/</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>slug<span class="token interpolation-punctuation punctuation">}</span></span><span class="token template-punctuation string"> : slug;
}
```

补充：astro 5.0新特性：

https://blog.csdn.net/gitblog_00592/article/details/152524702

<img width="856" height="425" alt="image" src="https://github.com/user-attachments/assets/7b03cde3-81fc-43c4-b134-ee45bd40416f" />



## 字体修改


字体修改：

- 字体分包工具：https://chinese-font.netlify.app/zh-cn/online-split
- 字体cdn：https://chinese-font.netlify.app/zh-cn/cdn/

```css
\src\styles\global.css
:root {
--font-mono: 'JetBrains Mono', 'JetBrainsMono', 'Fira Mono', 'Menlo', 'Consolas', monospace;
--font-cn: 'LXGW WenKai GB', 'Microsoft YaHei', 'PingFang SC', 'WenQuanYi Micro Hei', sans-serif;
}
h1, h2, h3, h4, h5, h6,span,figcaption,div{font-family: 'LXGW WenKai GB', sans-serif;}
li{font-family: 'JetBrains Mono', sans-serif;}
p {font-family: 'LXGW WenKai GB', sans-serif;font-size: 18px;}
/* 博客标题字体 */
a.absolute {
font-size: 22px;  
font-family: lxgw wenkai, sans-serif; 
}
```



## 文章过期提示



创建 \src\components\ExpiredBanner.astro 文件，复制如下代码

```astro
---
const { pubDatetime, expiryDays = 30 } = Astro.props;
const now = new Date();
const pubDate = new Date(pubDatetime);
// 计算整数天数
const daysDiff = Math.floor((now.getTime() - pubDate.getTime()) / (1000 * 3600 * 24));
// 是否过期
const isExpired = daysDiff > expiryDays;
{isExpired && (
<div class="expired-banner">
本文发布于 {pubDate.toLocaleDateString()}，距今已过去 {daysDiff} 天，内容可能已过期。
</div>
)}
```



在 mdx 文件引入。（需开启mdx支持：[astro - extending-markdown-config-from-mdx](https://docs.astro.build/en/guides/markdown-content/#extending-markdown-config-from-mdx)）

```md
---
title: 'asdfdg'
description: 'asdasd'
pubDatetime: 2025-06-01T00:00:00Z
slug: "20250601190253"  
---
我的文章标题
这是文章的内容……
import ExpiredBanner from '@/components/ExpiredBanner.astro'
<ExpiredBanner pubDatetime={frontmatter.pubDatetime} expiryDays={30} />
```


## 图标修改

`\src\constants.ts` 社交链接修改

图标网站：

* https://icon-sets.iconify.design
* https://icones.js.org

## Giscus评论

https://stackoverflow.com/questions/4425198/can-i-create-links-with-target-blank-in-markdown

[为你的 Astro 博客添加评论功能](https://liruifengv.com/posts/add-comments-to-astro/) + [how-to-integrate-giscus-comments](https://astro-paper.pages.dev/posts/how-to-integrate-giscus-comments/)


## js 插入

`\src\layouts\Layout.astro`

```
<script is:inline defer src="https://cloud.umami.is/script.js" data-website-id="e8dda199-cdc2-4bb5-8aeb-1a6bd156ada7"></script>
```


## http 调试


```
npm run build
cd dist/
http-server -p 4000
```

## page cms

https://app.pagescms.org/

在设置里，复制如下代码

```
media:
  input: public
  output: /
content:
  - name: posts
    label: Posts
    type: collection
    path: src/data/blog
    view:
      fields: [ title, draft, date ]
    fields:
      - { name: author, label: Author, type: string }
      - { name: pubDatetime, label: Date, type: date }
      - { name: modDatetime, label: Date, type: date }
      - { name: title, label: Title, type: string, required: true }
      - { name: ogImage, label: Title, type: string }
      - { name: slug, label: Slug, type: string }
      - { name: featured, label: Featured, type: boolean }
      - { name: draft, label: Draft, type: boolean }
      - { name: tags, label: Tags, type: string, list: true }
      - { name: description, label: Description, type: string }
      - { name: body, label: Body, type: rich-text }
  - name: about
    label: About page
    type: file
    path: src/pages/about.md
    fields:
      - { name: layout, type: string, hidden: true, default: "../layouts/AboutLayout.astro" }
      - { name: title, label: Title, type: string }
      - { name: body, label: Body, type: rich-text, options: { input: public/assets, output: /assets } }
```

## AstroPaper缓存策略与性能调优

https://blog.csdn.net/gitblog_00680/article/details/152506722

## 扩展

- astro umami 统计：https://github.com/yeskunall/astro-umami
- astro markdown 外链：https://odhyp.com/astro-opening-external-links
