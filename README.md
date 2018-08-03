
## Xcoding

> 专注于xcoding的网站。。。

[http://xcoding.tech](http://xcoding.tech)


本repo只用于提交md文件，发布后的静态文件全部保存在`public`目录下，这些文件直接上传到[`ubuntuvim.github.io`](http://xcoding.tech)项目中部署。


生成静态HTML命令如下：
```shell
hexo g
```

得到静态HTML之后直接通过如下命令发布到[xcoding.tech](http://xcoding.tech)。
```shell
hexo deploy
```


在此之前要先执行`npm install`安装依赖包。

所有的博客存放在`source/_posts`目录下。执行命令`hexo g`后这些md文件会转成静态`html`文件。
