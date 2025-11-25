---


---

<style>.token.pre.gfm *,.prism *{font-weight:inherit !important}.token.pre.gfm .token.comment,.token.pre.gfm .token.prolog,.token.pre.gfm .token.doctype,.token.pre.gfm .token.cdata,.prism .token.comment,.prism .token.prolog,.prism .token.doctype,.prism .token.cdata{color:#708090}.token.pre.gfm .token.punctuation,.prism .token.punctuation{color:#999}.token.pre.gfm .namespace,.prism .namespace{opacity:0.7}.token.pre.gfm .token.property,.token.pre.gfm .token.tag,.token.pre.gfm .token.boolean,.token.pre.gfm .token.number,.token.pre.gfm .token.constant,.token.pre.gfm .token.symbol,.token.pre.gfm .token.deleted,.prism .token.property,.prism .token.tag,.prism .token.boolean,.prism .token.number,.prism .token.constant,.prism .token.symbol,.prism .token.deleted{color:#905}.token.pre.gfm .token.selector,.token.pre.gfm .token.attr-name,.token.pre.gfm .token.string,.token.pre.gfm .token.char,.token.pre.gfm .token.builtin,.token.pre.gfm .token.inserted,.prism .token.selector,.prism .token.attr-name,.prism .token.string,.prism .token.char,.prism .token.builtin,.prism .token.inserted{color:#690}.token.pre.gfm .token.operator,.token.pre.gfm .token.entity,.token.pre.gfm .token.url,.token.pre.gfm .language-css .token.string,.token.pre.gfm .style .token.string,.prism .token.operator,.prism .token.entity,.prism .token.url,.prism .language-css .token.string,.prism .style .token.string{color:#a67f59}.token.pre.gfm .token.atrule,.token.pre.gfm .token.attr-value,.token.pre.gfm .token.keyword,.prism .token.atrule,.prism .token.attr-value,.prism .token.keyword{color:#07a}.token.pre.gfm .token.function,.prism .token.function{color:#dd4a68}.token.pre.gfm .token.regex,.token.pre.gfm .token.important,.token.pre.gfm .token.variable,.prism .token.regex,.prism .token.important,.prism .token.variable{color:#e90}.token.pre.gfm .token.important,.token.pre.gfm .token.bold,.prism .token.important,.prism .token.bold{font-weight:500}.token.pre.gfm .token.italic,.prism .token.italic{font-style:italic}.prism.language-markdown .token.bold,.prism.language-md .token.bold,.token.pre.gfm .language-markdown .token.bold,.token.pre.gfm .language-md .token.bold{font-weight:600 !important}code{background-color:rgba(0,0,0,0.05);border-radius:3px;padding:2px 4px}pre>code{background-color:rgba(0,0,0,0.05);display:block;padding:0.5em;-webkit-text-size-adjust:none;overflow-x:auto;white-space:pre}
</style>
<h2 id="坑点">坑点</h2>
<h3 id="npm-依赖报错：">npm 依赖报错：</h3>
<ol>
<li>删除 node_modules、package-lock.json、<code>npm cache clean --force &amp;&amp; npm install</code></li>
<li><a href="https://github.com/withastro/astro/issues/6830">https://github.com/withastro/astro/issues/6830</a></li>
</ol>
<h3 id="时区问题">时区问题</h3>
<p>npm dev 文章正常显示，但 build + http server 文章却没有按预期显示的原因：当 dev 状态的时候，不检查后面，如果是 build 状态就要检查发布时间了。</p><p>时间格式参考：</p><ul>
<li>2025-11-24 23:13:43+08:00</li>
<li>2025-03-20T03:15:57.792Z</li>
</ul>
<p>注意检查主题有关于配置时间、时区的配置文件。</p><h3 id="action">action</h3>
<p>根目录创建  <code>.nojekyll</code> 文件</p><h2 id="简要配置定位">简要配置定位</h2>
<ul>
<li><code>\src\config.ts</code> 博客资料页修改</li>
<li><code>\src\pages\index.astro</code> 欢迎页修改</li>
</ul>
<h2 id="固定url，文章移动到任意文件夹保持固定链接">固定URL，文章移动到任意文件夹保持固定链接</h2>
<p>URL效果： <code>localhost:4321/posts/{slug}</code></p><p>使用方式，全文替换如下文件</p><p><code>\src\pages\posts\[...slug]\index.astro</code></p><pre class="language-astro" tabindex="0"><code class="prism language-astro">---
import { getCollection } from "astro:content";
import PostDetails from "@/layouts/PostDetails.astro";
import { getPath } from "@/utils/getPath";
import getSortedPosts from "@/utils/getSortedPosts";

export async function getStaticPaths() {
  const posts = await getCollection("blog", ({ data }) =&gt; !data.draft);  // 获取所有非草稿文章
  const postResult = posts.map(post =&gt; ({
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
&lt;PostDetails post={post} posts={sortedPosts} /&gt;
</code></pre>
<p><code>\src\utils\getPath.ts</code></p><pre class="language-ts" tabindex="0"><code class="prism language-ts"><span class="token keyword">import</span> <span class="token punctuation">{</span> slugifyStr <span class="token punctuation">}</span> <span class="token keyword">from</span> <span class="token string">"./slugify"</span><span class="token punctuation">;</span>  <span class="token comment">// 引入 slugify 工具</span>

<span class="token comment">// 生成文章的 URL 路径</span>
<span class="token keyword">export</span> <span class="token keyword">function</span> <span class="token function">getPath</span><span class="token punctuation">(</span>id<span class="token operator">:</span> <span class="token builtin">string</span><span class="token punctuation">,</span> filePath<span class="token operator">:</span> <span class="token builtin">string</span> <span class="token operator">|</span> <span class="token keyword">undefined</span><span class="token punctuation">,</span> includeBase <span class="token operator">=</span> <span class="token boolean">true</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token comment">// 使用 slugifyStr 生成符合 URL 格式的 slug</span>
  <span class="token keyword">const</span> slug <span class="token operator">=</span> <span class="token function">slugifyStr</span><span class="token punctuation">(</span>id<span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">// 使用 id 生成 slug，确保符合 URL 格式</span>
  <span class="token builtin">console</span><span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>filePath<span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">// 这样使用后，警告就会消失</span>

  <span class="token comment">// 仅返回 slug，生成固定路径</span>
  <span class="token keyword">return</span> includeBase <span class="token operator">?</span> <span class="token template-string"><span class="token template-punctuation string">`</span><span class="token string">/posts/</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>slug<span class="token interpolation-punctuation punctuation">}</span></span><span class="token template-punctuation string">`</span></span> <span class="token operator">:</span> slug<span class="token punctuation">;</span>
<span class="token punctuation">}</span>

</code></pre>
<h2 id="字体修改">字体修改</h2>
<p>字体修改：</p><ul>
<li>字体分包工具：<a href="https://chinese-font.netlify.app/zh-cn/online-split">https://chinese-font.netlify.app/zh-cn/online-split</a></li>
<li>字体cdn：<a href="https://chinese-font.netlify.app/zh-cn/cdn/">https://chinese-font.netlify.app/zh-cn/cdn/</a></li>
</ul>
<p><code>\src\styles\global.css</code></p><pre><code>@import "../assets/fonts/JetBrainsMono/result.css";
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
</code></pre>
<h2 id="文章过期提示">文章过期提示</h2>
<p>创建 \src\components\ExpiredBanner.astro 文件，复制如下代码</p><pre><code>---
const { pubDatetime, expiryDays = 30 } = Astro.props;

const now = new Date();
const pubDate = new Date(pubDatetime);

// 计算整数天数
const daysDiff = Math.floor((now.getTime() - pubDate.getTime()) / (1000 * 3600 * 24));

// 是否过期
const isExpired = daysDiff &gt; expiryDays;
---

{isExpired &amp;&amp; (
  &lt;div class="expired-banner"&gt;
    本文发布于 {pubDate.toLocaleDateString()}，距今已过去 {daysDiff} 天，内容可能已过期。
  &lt;/div&gt;
)}
</code></pre>
<p>在 mdx 文件引入。（需开启mdx支持：<a href="https://docs.astro.build/en/guides/markdown-content/#extending-markdown-config-from-mdx">astro - extending-markdown-config-from-mdx</a>）</p><pre><code>---
title: 'asdfdg'
description: 'asdasd'
pubDatetime: 2025-06-01T00:00:00Z
slug: "20250601190253"  
---

# 我的文章标题

这是文章的内容……

import ExpiredBanner from '@/components/ExpiredBanner.astro'
&lt;ExpiredBanner pubDatetime={frontmatter.pubDatetime} expiryDays={30} /&gt;

</code></pre>
<h2 id="扩展">扩展</h2>
<ul>
<li>astro umami 统计：<a href="https://github.com/yeskunall/astro-umami">https://github.com/yeskunall/astro-umami</a></li>
<li>astro markdown 外链：<a href="https://odhyp.com/astro-opening-external-links">https://odhyp.com/astro-opening-external-links</a></li>
</ul>

