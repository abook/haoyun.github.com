---
layout: post
title: "Pandoc 及 MathJax 在 Octopress 中折腾纪实"
date: 2012-08-25 09:32
comments: true
categories: [octopress, geek, reprinting]
---

又是一晚上的折腾，把 [jekyll][jekyll] 默认的 markdown 语法解释器换成了更强大的 [pandoc][pandoc]，然后又修改了部分 CSS，作为一个狂热的 LaTeX 爱好者，自然要添加 [MathJax][mathjax] 支持。折腾是无止境的，但从另一个可能的角度来说，一次折腾，终身受益。

事实上，只要把 markdown 的解释器换成 pandoc，而 pandoc 支持 MathJax，所以一切都是自然的事情。但是，网上的教程自己实践起来总是要费一些功夫的。首先参考这篇文章 [为 Jekyll 装上瑞士军刀 Pandoc][jekyllpandoc]，其基本内容没有错误，但在我这里，即使已经通过自己的用户以及 root 用户都安装了 [pandoc-ruby][pr]，它却总是提示没有安装，非常无奈，最后发现只要修改网站根目录下的 `Gemfile`
即可，即给里面添加上

    gem 'pandoc-ruby', '~> 0.5.0'

这样，jekyll 就能找到 pandoc-ruby 了。另外也可以不用卸载在重装 jekyll，只需要按照上面文章里所述的原理修改 markdown.rb 文件即可。

<!--more-->

另外的一个问题是，如何配置 pandoc 的命令行参数，特别的，我想更改 MathJax 的 CDN
为豆瓣上的那个，服务器在国内毕竟快一些好一些。可是完全不懂 YAML 的语法，所以只能摸索，最可恨的网址里面含有下划线，jekyll 会把它处理成减号，经过不断的实验（没有搜到可用的资料），发现下面的两种方法是可行的。

    pandoc:
        extensions: [standalone, 'mathjax="https://d3eoax9i5htok0.cloudfront.net/mathjax/latest/MathJax.js?config=TeX-AMS-MML%5fHTMLorMML"']

或者

    pandoc:
        extensions:
           - standalone
           - mathjax=mathjax=https://d3eoax9i5htok0.cloudfront.net/mathjax/latest/MathJax.js?config=TeX-AMS-MML%5fHTMLorMML

折腾啊，我都不知道我是怎么尝试出来那个单引号，双引号，还有那个 `%5f` 的。

结果还没有完，上面的教程中说要使用参数 `--standalone`，这是让 pandoc 生成一个完整的页面，包含所需要的

    <script 
       src="https://d3eoax9i5htok0.cloudfront.net/mathjax/latest/MathJax.js?config=TeX-AMS-MML%5fHTMLorMML" type="text/javascript">
    </script>
而不仅是 `<body>` 中的内容，但这样生成的最终的页面代码是冗余的，不满意，所以
把那个 `<script>` 添加到 `_includes/custom/head.html`，这样就可以了。

故事不会这么快结束，不知道是那个地方的问题，MathJax 的右键菜单有问题：点击后菜单的背景是全屏的，尽管无伤大雅，但是总归不爽，经实验确定，应该不是 pandoc 的问题，可能是某个 CSS 或者某个 JS 导致的。<del>该停工了，暂时这样吧，有机会了再解解决。总之，可以较完美的现实数学公式了，这样博客的功能也就齐全了。</del>

既然大致确定了问题的所在，所以就开始行动了，首先检查一下网页加载了哪些 CSS，最终确定是一个叫做 `screen.css` 的样式表惹的祸。恼火的是，这是个压缩的单行 CSS 文件，难以阅读，所以请来 FireBugs——几个月前折腾浏览器时遇到的——非常简单，FireBug 就帮我把那个混乱的东西整理排版好了。然后分段删除，测试，删除，测试，最终，确定了是这几行

    body > div > div {
            background: url("/images/noise.png?1345858216") repeat scroll left top #F8F8F8;
                border-right: 1px solid #E0E0E0;
    }
惹的祸。接下来请出强大的 GREP

    # find . -name \* -type f -print | xargs grep "noise.png"
    ./base/_theme.scss:$noise-bg: image-url('noise.png') top left !default;
    ./base/_theme.scss:$nav-bg-front: image-url('noise.png') !default;
    ./base/_theme.scss:$footer-bg-front: image-url('noise.png') !default;
    ./custom/_colors.scss://$nav-bg-front: image-url('noise.png');
    ./custom/_colors.scss://$footer-bg-front: image-url('noise.png');
显然，问题出在 `_theme.scss` 这个文件上了。算是一个小小的冲突。修改一下就好了。

万恶的是，就在此时，我在想怎么修改的时候，发现了[这篇][octopresslatx]文章的最下面有一个链接，给我了所所遇到的问题的解决方案！折腾的意义在哪里？！
照着修改就好了。
    
给几个示例公式

**The Lorenz Equations**

$$
\begin{aligned}
\dot{x} & = \sigma(y-x) \\
\dot{y} & = \rho x - y - xz \\
\dot{z} & = -\beta z + xy 
\end{aligned}
$$

**The Cauchy-Schwarz Inequality**

$$
\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)
$$

**A Cross Product Formula**

$$
\mathbf{V}_1 \times \mathbf{V}_2=
\begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 
\end{vmatrix}
$$



PS:
---

折腾了两天，发现好多东西还是有很多问题，折腾的结果是：

* 发现了很多前在的问题。
* 找到了解决绝大部分问题的不优雅的方法。
* 放弃 pandoc，改投 [kramdown][kramdown]。


[jekyll]: http://jekyllrb.com/
[pandoc]: http://johnmacfarlane.net/pandoc/
[mathjax]: http://www.mathjax.org/
[jekyllpandoc]: http://yangzetian.github.com/2012/04/15/jekyll-pandoc/
[pr]: https://github.com/alphabetum/pandoc-ruby
[octopresslatex]: http://chen.yanping.me/cn/blog/2012/03/10/octopress-with-latex/
[kramdown]: http://kramdown.rubyforge.org/
