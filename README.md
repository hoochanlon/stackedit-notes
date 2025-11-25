# stackedit-notes
power by stackedit


# stackedit-notes
power by stackedit


# note

测试在线笔记

* https://stackeditpro.io
* https://preview-v2.docsify-this.net
* https://github.dev/hoochanlon/note/blob/main/README.md

坑点：

npm 依赖报错：

* 删除 node_modules、package-lock.json、` npm cache clean --force && npm install`
* https://github.com/withastro/astro/issues/6830

  
npm dev 与 build + http server 效果不一样。


> 当 dev 状态的时候，不检查后面，如果是 build 状态就要检查发布时间了，所以你的发布时间的格式看起来有点问题，可以看下其他的文章时间


* 2025-11-24 23:13:43+08:00
* 2025-03-20T03:15:57.792Z

---

\src\config.ts 博客资料页修改

\src\pages\index.astro 欢迎页修改

umami统计：https://github.com/yeskunall/astro-umami

字体修改：

- 字体分包工具：https://chinese-font.netlify.app/zh-cn/online-split
- 字体cdn：https://chinese-font.netlify.app/zh-cn/cdn/

\src\styles\global.css

```
@import "../assets/fonts/JetBrainsMono/result.css";
@import "../assets/fonts/lxgw-wenkai/result.css";
:root {
--font-mono: 'JetBrains Mono', 'JetBrainsMono', 'Fira Mono', 'Menlo', 'Consolas', monospace;
--font-cn: 'LXGW WenKai GB', 'Microsoft YaHei', 'PingFang SC', 'WenQuanYi Micro Hei', sans-serif;
}
h1, h2, h3, h4, h5, h6,span,figcaption,div{font-family: 'LXGW WenKai GB', sans-serif;}
li{font-family: 'JetBrains Mono', sans-serif;}
p {font-family: 'LXGW WenKai GB', sans-serif;font-size: 18px;}
/* 博客标题字体 /
a.absolute {
font-size: 22px;  / 修改字体大小 /
font-family: lxgw wenkai, sans-serif;  / 修改字体类型 */
}
```

---


URL  


C:\Users\hooch\Documents\GitHub\hoochanlon.github.io\src\pages\posts\[...slug]\index.astro


```
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
---
<PostDetails post={post} posts={sortedPosts} />

```

C:\Users\hooch\Documents\GitHub\hoochanlon.github.io\src\utils\getPath.ts

```
import { slugifyStr } from "./slugify";  // 引入 slugify 工具

// 生成文章的 URL 路径
export function getPath(id: string, filePath: string | undefined, includeBase = true) {
  // 使用 slugifyStr 生成符合 URL 格式的 slug
  const slug = slugifyStr(id);  // 使用 id 生成 slug，确保符合 URL 格式
  console.log(filePath);  // 这样使用后，警告就会消失

  // 仅返回 slug，生成固定路径
  return includeBase ? `/posts/${slug}` : slug;
}
```



