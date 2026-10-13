---
title: 环境搭建以及使用Ember.js创建第一个静态页面
tag:
  - Emberjs
  - Ember-Demo 
---

来源：[yoember.com](http://yoember.com/)  
作者：[Zoltan](http://Zoltan.nz)

**声明**：*本文的转载与翻译是经过作者认可的，再次感谢原作，如有侵权请给我留言，我会删除博文！！* 希望本系列教程能帮助更多学习Ember.js的初学者。



本篇将为读者介绍Ember项目开发环境的搭建，并创建一个静态页面。

## 安装Ember CLI

本教程使用的是`2.4.3`版本的Ember CLI工具集，如果你的是`1.13.8`版本在启动项目时会提示如下错误：
`
Future versions of Ember CLI will not support v5.9.1. Please update to Node 0.12 or io.js.
`
但是项目仍然可以正常访问，不过建议还是升级到`2.4.3`版本，省得出现未知错误不好解决。
升级命令：`npm install -g ember-cli@2.4.3`

查看`ember`命令是否安装成功，在终端或者控制台下输入下面的命令
```
ember -v
```
如果出现如下信息说明环境搭建成功。
```
version: 2.4.3
node: 5.9.1
npm: 2.13.4
os: darwin x64
```
如果你用的电脑不是Mac最后一行os有所不同，这个不要紧。第一行是`Ember CLI`的版本号，第二行是`node`的版本号，第三行是`npm`的版本号，最后一个是系统版本。

**注意**

如果你的执行`ember -v`得不到上述的版本信息也不要紧，仍然按照下面的教程新建项目新建完成项目之后再更行Ember CLI的版本，更新教程请参考[Could this be a shame in the making?](https://github.com/ember-cli/ember-cli/releases)，只需要根据**Project Update**部分更新项目即可，更新到最后一步`ember init`时候会如下确认信息，全部`y`即可。

```shell
? Overwrite .travis.yml? Yes, overwrite
? Overwrite .watchmanconfig? Yes, overwrite
? Overwrite README.md? Yes, overwrite
? Overwrite app/app.js? Yes, overwrite
? Overwrite app/index.html? Yes, overwrite
? Overwrite app/router.js? Yes, overwrite
? Overwrite bower.json? Yes, overwrite
? Overwrite ember-cli-build.js? Yes, overwrite
? Overwrite package.json? Yes, overwrite
? Overwrite tests/helpers/resolver.js? Yes, overwrite
? Overwrite tests/helpers/start-app.js? Yes, overwrite
? Overwrite tests/index.html? Yes, overwrite
```

更新过程可能还会出现如下选择版本的问题，请根据下面例子选择：
```shell
Installed packages for tooling via npm.
  conflict Unable to find suitable version for qunit-notifications
    1) qunit-notifications ~0.0.6
    2) qunit-notifications ~0.1.0
? Answer 2
  conflict Unable to find suitable version for ember
    1) ember >= 1.8.1 < 2.0.0
    2) ember >=1.4 <2
    3) ember > 1.5.0-beta.3
    4) ember ~2.4.3
    5) ember >=1.4
? Answer 4
Installed browser packages via Bower.
```
最后验证是否更新成功，执行`ember -v`会得到如下版本信息：
```
ubuntuvimdeMacBook-Pro:library-app ubuntuvim$ ember -v
ember-cli: 2.4.3
node: 5.9.1
os: darwin x64
```

更多有关开发环境的详细介绍请看[www.ember-cli.com](http://ember-cli.com/user-guide)。

## 创建一个新项目

安装好开发环境之后，直接使用[Ember CLI](http://ember-cli.com/user-guide)命令创建新项目。下面是创建命令：
```
ember new library-app
```
等待命令执行完成，安装过程需要下载所必须的npm插件，跟网络有关系，请耐心等待。

## 运行项目

等待项目创建完成之后就可以直接使用命令运行项目了，首先进入项目目录下，然后执行ember cli命令运行项目。
```
//  进入项目目录下
cd library-app
//  执行启动命令
ember server
```
*`//`的内容为注释，请直接忽略。*

等待启动完毕后，打开浏览器执行[http://localhost:4200](http://localhost:4200)，如果能在页面上看到**Welcome to Ember**说明项目创建成功。并且可以在浏览器控制台上看到如下图的日志信息：

![日志信息](/image/blog-image/55.png)

## 开启调试模式

在开发阶段最好是把打开调试模式，开启之后可以在浏览器的控制台下看到ember项目执行过程的相关信息，有助于发现问题。
修改`library-app/config/environment.js`文件的内容，在下面代码段中增加配置：
```
// ……
if (environment === 'development') {
  // ENV.APP.LOG_RESOLVER = true;
  ENV.APP.LOG_ACTIVE_GENERATION = true;
  ENV.APP.LOG_TRANSITIONS = true;
  ENV.APP.LOG_TRANSITIONS_INTERNAL = true;
  ENV.APP.LOG_VIEW_LOOKUPS = true;
}
//……
```
重启项目（按`Ctrl+C`终止在执行`ember servere`），必须重启才能其效果，可以在浏览器控制台看到了很多的日志信息。比如下图

![开启日志模式截图](/image/blog-image/56.png)


## 添加Bootstrap和Sass到项目中

为了美化项目界面引入[Bootstrap](http://www.bootcss.com/)，这两个插件的安装也是直接使用Ember CLI命令安装，命令如下：
```
ember install ember-cli-sass
ember install ember-cli-bootstrap-sassy
```
等待安装完成之后可以在项目目录下的`pachage.json`和`bower.json`看到这两个插件的配置信息。
```
//  bower.json
"bootstrap-sass": "^3.3.6"
// package.json
"ember-cli-sass": "5.3.1"
```

**在项目下增加样式文件**

创建文件`library-app/app/styles/app.scss`，如果项目已经存在文件`library-app/app/styles/app.css`则重命名为`app.scss`，样式会被Ember CLI引入到项目中。
在文件中增加如下内容：
```css
@import "bootstrap";
```
使用快捷键`Ctrl+C`关闭在用命令`ember server`启动项目。如果终端没出现错误说明配置是正确的。那么请继续往下看！！

## 创建项目导航条

在前面引入的了[Bootstrap](http://www.bootcss.com/)之后我们就可以在页面中直接使用了，并且不需要再在页面上引入相关的`css`和`js`文件。
打开文件`library-app/app/templates/application.hbs`，清空原有代码再添加如下代码：
```html
<div class="container">
    {{partial 'navbar'}}
    {{outlet}}
</div>
```
Ember.js项目的页面使用的是[Handlebarsjs](http://handlebarsjs.com/)模板，`{% raw %}{{}}{% endraw %}`是模板的语法。在Ember.js的官方参考教程中有一章是专门介绍如何使用Handlebarsjs模板的，或者根据[Ember.js 入门指南之八handlebars基础](http://blog.ddlisting.com/2016/03/18/ember-js-ru-men-zhi-nan-zhi-ba-handlebarsji-chu/)学习。
在上述代码中`{% raw %}{{partial}}{% endraw %}`是一个[ember helper](https://guides.emberjs.com/v1.13.0/templates/rendering-with-helpers/)可以用于调用模板，这里就是调用了模板`navbar`，不过这个功能在`2.4`的参考文档中移除了可以在`1.13.0`的文档中看到，更多有关信息请看参考网址。
代码中`{% raw %}{{outlet}}{% endraw %}`也是一个helper，但是这个是一个特殊的helper，你可以把这个helper理解为一个占位符。所有子模板都会渲染到`{% raw %}{{outlet}}{% endraw %}`所在的位置。更多信息请看[Ember.js 入门指南之十四番外篇，路由、模板的执行、渲染顺序](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-si-fan-wai-pian-lu-you-mo-ban-de-zhi-xing-xuan-ran-shun-xu/)。

**创建一个模板**

仍然是使用命令创建模板。
```
ember g template navbar
```
等待命令执行完毕之后可以看到`library-app/app/templates/navbar.hbs`这个文件。下面在文件中增加一个导航条。
```html
<nav class="navbar navbar-inverse">
  <div class="container-fluid">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#main-navbar">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      {{#link-to 'index' class="navbar-brand"}}Library App{{/link-to}}
    </div>

    <div class="collapse navbar-collapse" id="main-navbar">
      <ul class="nav navbar-nav">
            {{#link-to 'index' tagName="li"}}<a href="">Home</a>{{/link-to}}
      </ul>
    </div><!-- /.navbar-collapse -->
  </div><!-- /.container-fluid -->
</nav>
```
代码中`{% raw %}{{link-to}}{% endraw %}`是Handlebars模板的标签，在第一个`{% raw %}{{link-to}}{% endraw %}`标签中`index`是一个路由的名字，模板被编译之后这个标签就转成一个普遍HTML标签的`<a>`，如果你想指定编译之后的标签名请使用属性`tagName`指定，比如上述代码的第二个`link-to`标签，在后面的文章中会使用组件（`component`）重构这个标签。
*为了美化界面在页面的顶部加了css的填充，修改样式文件`app.scss`。*
```css
@import "bootstrap";

body {
    padding-top: 20px;
}
```
等待项目重启完成，可以在页面上看到黑色的导航条，好像我们并且没有在任何地方使用这个模板`navbar`，为何能在首页上显示呢？？其实我们已经在`application.hbs`中调用了！在这个模板中有这样一句代码`{% raw %}{{partial 'navbar'}}{% endraw %}`，在此根据模板名调用了模板`navbar`。如果删除了`application.hbs`中的`{% raw %}{{partial}}{% endraw %}`界面上就什么都不显示了！请读者自行实验。

## 创建关于界面并在导航菜单上增加一个菜单项

同样的，使用Ember CLI命令创建一个路由（route），有关路由的信息可以查看官方参考文档或者直接看教程[Ember.js 入门指南之二十路由定义](http://blog.ddlisting.com/2016/03/25/ember-js-ru-men-zhi-nan-zhi-er-shi-lu-you-ding-yi/)，文章上有详细的介绍，欢迎阅读！执行下面的命令创建路由，创建路由的过程中会同时创建路由对应的模板，所以执行一个命令会得到2个文件：`app/templates/about.hbs`、`app/routes/about.js`，同时会在`app/router.js`中app/自动增加一条路由配置语句`this.route('about');`。然后在模板`about.hbs`中增加一些信息：
```html
{{! app/templates/about.hbs }}
<h1>About Page</h1>
```
等待项目重启完成，执行[http://localhost:4200/about](http://localhost:4200/about)可以看到刚刚在模板`about.hbs`中增加的信息，如下截图。

![结果截图](/image/blog-image/81.png)

但是"About Page"怎么会显示在导航条下方呢？好像并没有指定啊，也没有想前面那样使用表达式`{% raw %}{{partial}}{% endraw %}`调用模板，有关这个内容的介绍请看[Ember.js 入门指南之十四番外篇，路由、模板的执行、渲染顺序](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-si-fan-wai-pian-lu-you-mo-ban-de-zhi-xing-xuan-ran-shun-xu/)，还记得在主模板`application.hbs`中的`{% raw %}{{outlet}}{% endraw %}`吗？除了`application.hbs`之外的所有模板都是子模板，子模板会自动渲染到父模板的`{% raw %}{{outlet}}{% endraw %}`上。但是是如何触发显示的呢？很简单，因为我们访问了`about`这个路由，路由会自动根据名字查找到同名的模板并显示（Ember默认规则）。

再创建一个模板`index`，仍然是使用Ember CLI命令创建，执行命令：`ember g template index`，得到模板后再模板内添加一些内容：
```html
{{! app/templates/index.hbs}}
<h1>Home Page</h1>
```
然后执行[http://localhost:4200/](http://localhost:4200/)，神奇的事情发生了，可以直接看到模板`index`的内容，并且并没有访问[http://localhost:4200/index](http://localhost:4200/index)。这又是为什么呢？请看[Ember.js 入门指南之二十路由定义](http://blog.ddlisting.com/2016/03/25/ember-js-ru-men-zhi-nan-zhi-er-shi-lu-you-ding-yi/)中关于`index`路由的解释。简单讲，`index`路由就是每个路由默认首页路由，不需要手动创建，这个路由对应的URL是`/`，当你执行[http://localhost:4200/](http://localhost:4200/)时候实际就是执行[http://localhost:4200/index](http://localhost:4200/index)然后渲染的模板就是`index.hbs`，所以就得到界面显示的效果。
然后在导航栏上在添加一个链接，最后得打如下代码（前后部分代码省略）：
```html
<ul class="nav navbar-nav">
            {{#link-to 'index' tagName="li"}}<a href="">Home</a>{{/link-to}}
            {{#link-to 'about' tagName="li"}}<a href="">About</a>{{/link-to}}
      </ul>
```
等待项目重启完成，可以看到导航栏上多了一项，并且点击“Home”和“About”看到显示不同的内容。效果如下图：

![效果图](/image/blog-image/81.png)

到此教程第一篇介绍完毕，如果你看过官方参考文件或者是看过[ember teach](http://blog.ddlisting.com/)上的教程理解起来应该是没难度的！多一份耐心就多一份收获。

## 家庭作业

最后给你留了一份作业，想学好就必须要动手实践才行啊！！！

作业内容：

1. 创建一个名为`contact`的路由和模板
2. 在导航菜单上增加一个菜单项"Contact"，并且点击这个菜单项看到的是模板`contact.hbs`的内容。

<br>
为了照顾懒人我把完整的代码放在[GitHub](https://github.com/ubuntuvim/library-app)上，如有需要请参考参考。博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
