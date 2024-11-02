---
layout:     post

title:      "vuepress配置博客支持数学公式"
# description: "用数值的方法求解热传导方程，使用最简显格式进行推导。"
author:     "谢文进"
date:       2022-03-09
# image: "/img/2021-03-11-eluer-method/background.jpg"
published: 2022-03-09 
tags:
     - 博客
     - 数学公式
     - vuepress

categories: [ Tech ]
URL: "/2022/03/09/mathEquationBlog/"
---

主要参考[vuepress-theme-reco设置math公式](https://www.1024sou.com/article/642755.html)。
## 步骤

1. 定位到博客的根目录下，安装库
```
npm i markdown-it-texmath
```
2. 安装`markdown-it`

```
npm install markdown-it --save
```

3. 安装`katex`

```
npm install katex
```

4. 修改`.vuepress/config.js`文件

```javascript
markdown: {
lineNumbers: true,
anchor: { permalink: false },
toc: {includeLevel: [1,2]},
extendMarkdown: md => {
md.use(require('markdown-it-texmath'))
}
```

`head`部分添加下面的代码：

```javascript
['link', {"rel":'stylesheet', "href":'https://cdn.jsdelivr.net/npm/katex@0.15.1/dist/katex.min.css'}],
['link', {"rel":'stylesheet', "href":'https://gitcdn.xyz/cdn/goessner/markdown-it-texmath/master/texmath.css'}],
['script', {"src": 'https://github.com/markdown-it/markdown-it/blob/master/bin/markdown-it.js'}],
['script', {"src": 'https://gitcdn.xyz/cdn/goessner/markdown-it-texmath/master/texmath.js'}],
['script', {"src": 'https://cdn.jsdelivr.net/npm/katex@0.15.1/dist/katex.min.js'}]
```

5. 更新提交后，就大功告成了。
