# hello-world
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub

hexo new [layout] <title>
hexo new photo "My Gallery"
hexo new "Hello World" --lang tw



以上是文章摘要 <!--more--> 以下是余下全文




---
title: hexo备忘
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).
<!--more-->
## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```
{% codeblock %}
alert('Hello World!');
{% endcodeblock %}

{% codeblock Array.map %}
array.map(callback[, thisArg])
{% endcodeblock %}

{% codeblock _.compact http://underscorejs.org/#compact Underscore.js %}
_.compact([0, 1, false, 2, '', 3]);
=> [1, 2, 3]
{% endcodeblock %}

<!--234323523-->

插入 source 文件夹内的代码文件。
{% include_code [title] [lang:language] path/to/file %}

引用其他文章的链接。
{% post_path slug %}
{% post_link slug [title] %}




{% img "/uploads/head.jpeg" "tupian" %}

More info: [Writing](https://hexo.io/docs/writing.html)
### Run server

``` bash
$ hexo server
```
<!--more-->
More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)


{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}

{% codeblock %}
alert('Hello World!');
{% endcodeblock %}

To embed an iframe:
{% iframe url [width] [height] %}

Inserts an image with specified size.
{% img [class names] "/uploads/head.jpeg" [200] [100] ["title" ["alt"]] %}


