---
title: 发布项目，加入CRUD功能
tag:
  - Emberjs
  - Ember-Demo 
---

来源：[yoember.com](http://yoember.com/)  
作者：[Zoltan](http://Zoltan.nz)

**声明**：_本文的转载与翻译是经过作者认可的，再次感谢原作，如有侵权请给我留言，我会删除博文！！_ 希望本系列教程能帮助更多学习Ember.js的初学者。


接着前面三篇：

1. [环境搭建以及使用Ember.js创建第一个静态页面](http://xcoding.tech/2016/03/30/Ember-Demo/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%BB%A5%E5%8F%8A%E4%BD%BF%E7%94%A8Ember.js%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%9D%99%E6%80%81%E9%A1%B5%E9%9D%A2/)
2. [引入计算属性、action、动态内容](http://xcoding.tech/2016/03/31/Ember-Demo/%E5%BC%95%E5%85%A5%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7%E3%80%81action%E3%80%81%E5%8A%A8%E6%80%81%E5%86%85%E5%AE%B9/)
3. [模型，保存数据到数据库](http://xcoding.tech/2016/03/31/Ember-Demo/%E6%A8%A1%E5%9E%8B%EF%BC%8C%E4%BF%9D%E5%AD%98%E6%95%B0%E6%8D%AE%E5%88%B0%E6%95%B0%E6%8D%AE%E5%BA%93/)

## 应用发布

**发布方式一**

发布的详细教程请看[guide on firebase](https://www.firebase.com/docs/web/libraries/ember/guide.html#section-ember-deploy)。执行如下命令发布项目。
```
npm install -g firebase-tools
ember build --prod
firebase login
firebase init
```
执行命令过程需要输入一个public的目录，输入`dist`后按`enter`。更新`firebase.json`的内容。
```js
{
  "firebase": "YOUR-APP-NAME",
  "public": "dist",
  "rewrites": [{
    "source": "**",
    "destination": "/index.html"
  }]
}
```
遗憾的是在我电脑上一直提示没有`firebase`命令，即使我已经安装了这个插件也不行。

**发布方式二**

由于上述方式无法发布想到到firebase，所以使用最原始的发布方式，使用`ember`命令打包项目。然后自己把项目部署到服务器上。

1. 打包项目
打包项目使用命令`ember build --prod`，等到命令执行完毕后再项目的`dist`目录下的所有文件即使打包后的项目文件。
2. 复制打包后的文件到服务器上
得到打包后的文件后可以直接把这些文件复制到服务器上运行，比如复制到tomcat的`webapps`目录下。
3. 运行项目
复制到服务器之后启动服务器，直接访问：[http://localhost:8080](http://localhost:8080)

## 增加删除、修改功能

修改项目的library列表页面，增加删除和修改功能。
```html
<!-- app/templates/libraries/index.hbs -->
<h2>List</h2>
<div class="row">
  {{#each model as |library|}}
    <div class="col-md-4">
      <div class="panel panel-default library-item">
          <div class="panel-heading">
              <h3 class="panel-title">{{library.name}}</h3>
          </div>
          <div class="panel-body">
              <p>Address: {{library.address}}</p>
              <p>Phone: {{library.phone}}</p>
          </div>
          <div class="panel-footer text-right">
              {{#link-to 'libraries.edit' library.id class='btn btn-success btn-xs'}}Edit{{/link-to}}
              <button class="btn btn-danger btn-xs" {{action 'deleteLibrary' library}}>Delete</button>
          </div>
      </div>
    </div>
  {{/each}}
</div>
```
相比原来的代码增加了一个连接和一个按钮，分别用于编辑和删除library信息。相对于需要增加一个路由`libraries/edit`和一个处理的动作`{% raw %}{{action 'deleteLibrary'}}{% endraw %}`。
如果此时运行[http://localhost:4200/libraries](http://localhost:4200/libraries)会出现错误，因为还没定义路由`libraries/edit`和`action`。别急，先一步步来，下面先增加一些css样式。

```css
# app/styles/app.scss
@import 'bootstrap';

body {
  padding-top: 20px;
}

html {
  overflow-y: scroll;
}

.library-item {
  min-height: 150px;
}
```

## 创建路由`libraries/edit`和路由对应的模板

简单起见直接使用[Ember CLI](http://ember-cli.com/user-guide)命令创建，就不手动创建了。执行命令：`ember g route libraries/edit`创建路由和路由对应的模板。
创建完成之后还需要手动修改`app/router.js`文件，内容如下：
```js
// app/router.js

import Ember from 'ember';
import config from './config/environment';

var Router = Ember.Router.extend({
  location: config.locationType
});

Router.map(function() {

  this.route('about');
  this.route('contact');

  this.route('admin', function() {
    this.route('invitation');
    this.route('contact');
  });

  this.route('libraries', function() {
    this.route('new');
    // :library_id是一个动态段，会根据实际的URL变化
    this.route('edit', { path: '/:library_id/edit' });
  });
});

export default Router;
```

注意`this.route('edit', { path: '/:library_id/edit' });`这行代码的设置。与普通的路由稍有不同这里增加了一个参数，并且参数内使用`path`设定路由渲染之后`edit`会被`/:library_id/edit`替换。
编译、渲染之后的URL格式为`http://example.com/libraries/1234/edit`其中`:library_id`这是一个动态段，这个URL例子中动态段`library_id`的值就是`1234`，并且可以在路由类中获取这个动态段的值。
更多有关动态段的介绍请看[Ember.js 入门指南之十三{{link-to}} 助手](http://blog.ddlisting.com/2016/03/22/ember-js-ru-men-zhi-nan-zhi-shi-san-link-to/)或者[Dynamic Segments](https://guides.emberjs.com/v2.5.0/routing/defining-your-routes/#toc_dynamic-segments)。

配置完路由之后修改路由`libraries/edit.js`的代码。
```js
// app/routes/libraries/edit.js
import Ember from 'ember';

export default Ember.Route.extend({

  model(params) {
    // 获取动态段library_id的值
    return this.store.findRecord('library', params.library_id);
  },

  actions: {

    saveLibrary(newLibrary) {
      newLibrary.save().then(() => this.transitionTo('libraries'));
    },

    willTransition(transition) {

      let model = this.controller.get('model');

      if (model.get('hasDirtyAttributes')) {
        let confirmation = confirm("Your changes haven't saved yet. Would you like to leave this form?");

        if (confirmation) {
          model.rollbackAttributes();
        } else {
          transition.abort();
        }
      }
    }
  }
});
```
代码`this.store.findRecord('library', params.library_id);`的意思是根据模型的`id`属性值查询某个记录，其中`library_id`就是动态段的值，这个值是Ember解析URL得到的。正如前面所说：`http://example.com/libraries/1234/edit`这个URL动态段的值就是`1234`。
Ember会自动根据URL的格式解析得到。并且可以在路由类中获取。默认情况下动态段的值是数据的`id`值。代码中的另外两个方法`saveLibrary()`和`willTransition()`在前一篇文章[模型，保存数据到数据库](http://blog.ddlisting.com/2016/04/09/mo-xing-bao-cun-shu-ju-dao-shu-ju-ku/)已经介绍过，在此不再赘述。
方法`willTransition()`的作用就是：当用户修改了数据之后没有点击保存就离开页面时会提示用户是否确认不保存就离开页面！通过控制器中的属性`hasDirtyAttributes`判断页面的值是否发生了变化。方法`rollbackAttributes()`会重置`model`中的值。方法`abourt()`可以阻止路由的跳转，有关路由的跳转请看[Ember.js 入门指南之二十四终止与重试路由跳转](http://blog.ddlisting.com/2016/03/25/ember-js-ru-men-zhi-nan-zhi-er-shi-si-zhong-yu-yu-zhong-shi-lu-you-zhuan-huan/)。从`new.hbs`复制代码到`edit.hbs`，然后在稍加修改。
```html
<!-- app/templates/libraries/edit.hbs -->

<h2>Edit Library</h2>

<div class="form-horizontal">
    <div class="form-group">
        <label class="col-sm-2 control-label">Name</label>
        <div class="col-sm-10">
          {{input type="text" value=model.name class="form-control" placeholder="The name of the Library"}}
        </div>
    </div>
    <div class="form-group">
        <label class="col-sm-2 control-label">Address</label>
        <div class="col-sm-10">
          {{input type="text" value=model.address class="form-control" placeholder="The address of the Library"}}
        </div>
    </div>
    <div class="form-group">
        <label class="col-sm-2 control-label">Phone</label>
        <div class="col-sm-10">
          {{input type="text" value=model.phone class="form-control" placeholder="The phone number of the Library"}}
        </div>
    </div>
    <div class="form-group">
        <div class="col-sm-offset-2 col-sm-10">
            <button type="submit" class="btn btn-default" {{action 'saveLibrary' model}}>Save changes</button>
        </div>
    </div>
</div>
```
等待项目重启完成，进入到修改界面，任意修改界面上的数据，不点击保存然后任意点击其他链接会弹出提示，询问你是否确认离开页面。操作步骤如下截图。

![library主页](http://blog.ddlisting.com/content/images/2016/04/162-2.png)

![library修改页面](http://blog.ddlisting.com/content/images/2016/04/163.png)

**注意**：看浏览器的URL。首页模板代码`{% raw %}{{#link-to 'libraries.edit' library.id class='btn btn-success btn-xs'}}Edit{{/link-to}}{% endraw %}`中的路由`libraries.edit`渲染之后会得到形如`libraries/xxx/edit`的URL，其中`xxx`就是动态段的值。

![修改name的值，然后点击菜单的Home](http://blog.ddlisting.com/content/images/2016/04/164.png)

![library未保存离开页面提示](http://blog.ddlisting.com/content/images/2016/04/165.png)

弹出提示信息。如果点击取消会停留在当前页面，如果选中确定会跳转到首页（因为我点击的是菜单的Home）。

![修改后点击保存](http://blog.ddlisting.com/content/images/2016/04/166.png)

成功保存了修改的内容。到此实现了修改功能。

## 实现删除功能

删除功能比修改更加简单，直接在方法`deleteLibrary`里根据`id`属性值删除数据即可。`id`属性值已经在模板页面作为参数传递到方法中。直接获取即可。
```js
// app/routes/libraries/index.js

import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.store.findAll('library');
  },
  actions: {
      // 删除一个library记录
      deleteLibrary(library) {  //参数library已经在模板中传递过来
      let confirmation = confirm('Are you sure?');

      if (confirmation) {
        library.destroyRecord();
      }
    }
  }
});
```
模板中是这样调用删除方法的`<button class="btn btn-danger btn-xs" {{action 'deleteLibrary' library}}>Delete</button>`，看到参数`library`了吧，这个参数就是一个`library`模型对象。
可以直接调用方法`destroyRecord()`实现删除数据。

![点击删除按钮](http://blog.ddlisting.com/content/images/2016/04/167.png)

选中确定之后删除就会立刻删除，列表上的数据也会动态更新。

## 家庭作业

参照library的功能实现contact的删除与修改。

#### 新建路由和模板

```
ember g route admin/contact/edit
ember g template admin/contact/index
```

#### 修改router.js，增加配置

```js
// app/router.js

this.route('admin', function() {
  this.route('invitation');
  this.route('contact', function() {
    this.route('edit', { path: '/:contact_id/edit' });
  });
});
```
省略其他内容，仅仅列出修改部分。

#### 复制`admin/contact.hbs`的内容到`admin/contact/index.hbs`，然后空`admin/contact.hbs`再在文件内添加`{% raw %}{{outlet}}{% endraw %}`

`admin/contact.hbs`
```html
<!-- app/templates/admin/contact.hbs -->
{{outlet}}
```
`admin/contact/index.hbs`

```html
{{! app/templates/admin/contact/index.hbs}}

<h1>Contacts</h1>

<table class="table table-bordered table-striped">
    <thead>
      <tr>
          <th>ID</th>
          <th>E-mail</th>
          <th>Message</th>
          <th>Operation</th>
      </tr>
    </thead>
    <tbody>
    {{#each model as |contact|}}
        <tr>
            <th>{{contact.id}}</th>
            <td>{{contact.email}}</td>
            <td>{{contact.message}}</td>
            <td>
                {{#link-to 'admin.contact.edit' contact.id class='btn btn-success btn-xs'}}Edit{{/link-to}}
                <button class="btn btn-danger btn-xs" {{action 'deleteContact' contact}}>Delete</button>
            </td>
        </tr>
    {{/each}}

    </tbody>
</table>
```
增加删除、修改按钮。

#### 复制`app/templates/contact.hbs`到`admin/contact/edit.hbs`并做修改

admin/contact/edit.hbs
```js
{{! app/templates/admin/contact/edit.hbs}}

<div class="col-md-6 col-xs-6">
   <form>
     <div class="form-group">
         <label for="exampleInputEmail1">Email address</label>
         {{input type="email" value=model.email class="form-control col-sm-6 col--6" placeholder="Please type your e-mail address." autofocus="autofocus"}}
     </div>
     <div class="form-group">
         <label for="exampleInputPassword1">Your message</label>
         {{textarea class="form-control" placeholder="Your message. (At least 5 characters.)" rows="7" value=model.message}}
     </div>

     <button class="btn btn-primary" disabled={{model.isDisabled}} {{action 'saveContact' model}}>Save</button>
     {{#link-to 'admin.contact' class="btn btn-default"}}Return{{/link-to}}
   </form>
</div>
```

#### 修改`routes/context.js`

```js
// app/routes/contact.js

import Ember from 'ember';

export default Ember.Route.extend({
    model: function() {
        return this.store.findAll('contact');
    },
    actions: {
        deleteContact: function(contact) {
            let confirmation = confirm('Are you sure?');

            if (confirmation) {
              contact.destroyRecord();
            }
        }
    }
});
```

#### 修改`app/routes/admin/contact/edit.js`

```js
// app/routes/admin/contact/edit.js

import Ember from 'ember';

export default Ember.Route.extend({

  model(params) {
    // 获取动态段library_id的值
    return this.store.findRecord('contact', params.contact_id);
  },

  actions: {

    saveContact(newContact) {
      newContact.save().then(() => this.transitionTo('admin.contact'));
    },

    willTransition(transition) {

      let model = this.controller.get('model');

      if (model.get('hasDirtyAttributes')) {
        let confirmation = confirm("Your changes haven't saved yet. Would you like to leave this form?");

        if (confirmation) {
          model.rollbackAttributes();
        } else {
          transition.abort();
        }
      }
    }
  }
});
```

运行结果不再截图列出，请读者自行试验。

<br>
为了照顾懒人我把完整的代码放在[GitHub](https://github.com/ubuntuvim/library-app)上，可以拿来做参照。博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
