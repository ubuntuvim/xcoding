---
title: 引入计算属性、action、动态内容
tag:
  - Emberjs
  - Ember-Demo 
---

来源：[yoember.com](http://yoember.com/)  
作者：[Zoltan](http://Zoltan.nz)

**声明**：_本文的转载与翻译是经过作者认可的，再次感谢原作，如有侵权请给我留言，我会删除博文！！_ 希望本系列教程能帮助更多学习Ember.js的初学者。

接着前面一篇：

1. [环境搭建以及使用Ember.js创建第一个静态页面](http://xcoding.tech/2016/03/30/Ember-Demo/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%BB%A5%E5%8F%8A%E4%BD%BF%E7%94%A8Ember.js%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%9D%99%E6%80%81%E9%A1%B5%E9%9D%A2/)


## 美化主页，增加邮件输入框

在主页中增加一个[Bootstrap](http://www.bootcss.com)的`jumbotron`，在这个`jumbotron`组件中增加一个`input`输入框和一个`button`按钮。


### 在首页index.hbs中增加静态HTML代码

为了界面的美化在HTML代码中使用了很多[Bootstrap](http://www.bootcss.com)的样式，更多有关Bootstrap的使用请自行学习。
```html
<div class="jumbotron text-center">
    <h1>Coming Soon</h1>

    <br/><br/>

    <p>Don't miss our launch date, request an invitation now.</p>

    <div class="form-horizontal form-group form-group-lg row">
        <div class="col-xs-10 col-xs-offset-1 col-sm-6 col-sm-offset-1 col-md-5 col-md-offset-2">
          <input type="email" class="form-control" placeholder="Please type your e-mail address." autofocus="autofocus"/>
        </div>
        <div class="col-xs-10 col-xs-offset-1 col-sm-offset-0 col-sm-4 col-md-3">
            <button class="btn btn-primary btn-lg btn-block">Request invitation</button>
        </div>

    </div>

    <br/><br/>
</div>
```
等待项目重启完成，可以在首页看到如下效果页面：

![首页效果截图](/content/images/2016/03/90.png)

## 计算属性

计算属性简单讲它就是一个特殊点的JS函数。如果你看过[Ember.js 入门指南之三计算属性（compute properties）](http://blog.ddlisting.com/2016/03/17/ember-js-ru-men-zhi-nan-ji-suan-shu-xing-compute-properties/)相信使用起来会比较简单，再次不过多介绍。

### 计算属性使用

下面几点需求可以通过计算属性去实现：

1. 当输入框的为空时按钮“Request invitation”不可用
2. 当输入的邮箱号码格式不正确时按钮“Request invitation”不可用
3. 点击按钮“Request invitation”之后显示响应信息
4. 数据保存完成之后情况邮箱输入框的内容

### isDisabled属性

既然介绍了计算属性那么应该拿来用了！我们使用属性`isDisabled`控制按钮“Request invitation”是否可用。在`button`标签上增加一个HTML属性`disabled`，这个HTML属性决定了按钮`button`是否可用。当HTML属性`disabled=true`时按钮不可用，当HTML属性`disabled=false`时按钮可用，那么如何控制这个值是`true`还是`false`呢？别忘了在Handlebars模板中可以直接使用`{% raw %}{{}}{% endraw %}`表达式获取属性的值，下面修改模板`index.hbs`，在标签`button`中增加属性`disabled`的设置：
```html
<button class="btn btn-primary btn-lg btn-block" disabled={{isDisabled}}>Request invitation</button>
```
那要在哪里控制`isDisabled`的值呢？目前有2种方式，第一是在路由`route`中控制这个值，另外一种是在控制器`controller`中控制这个属性的值。有关路由的信息在前一篇已经简单介绍过，或者看[Ember.js 入门指南之二十路由定义](http://blog.ddlisting.com/2016/03/25/ember-js-ru-men-zhi-nan-zhi-er-shi-lu-you-ding-yi/)学习。与路由同理，每个模板都对应有一个同名的控制器`controller`，如果你学习过MVC模式那么你应该很清楚什么是控制器，Ember中的控制器作用于MVC模式中的控制器相似，不过需要注意的是从`Ember 3.0`之后控制器将不再支持，所以呢！会在后面用组件替代控制器，官方也是这么推荐的！更多有关控制器的介绍请看[Controller Introduction](https://guides.emberjs.com/v2.4.0/controllers/)。
同样的我们仍然是使用Ember CLI创建控制器，命令如下：
```
ember g controller index
```
创建好控制器之后，在控制器内添加设置属性`isDisabled`的代码：
```js
// app/controller/index.js

import Ember from 'ember';

export default Ember.Controller.extend({
    isDisabled: true
});
```
等待项目重启完毕，可以看到按钮是不可用，如果你把属性`isDisabled`设置为`false`那么按钮是可用的。

## 计算属性与观察者

计算属性和观察者是Ember非常重要的特性。更多有关它们的特性请看下面的文章：

1. [Ember.js 入门指南之三计算属性](http://blog.ddlisting.com/2016/03/17/ember-js-ru-men-zhi-nan-ji-suan-shu-xing-compute-properties/)
2. [计算属性官方参考文档](https://guides.emberjs.com/v2.4.0/object-model/computed-properties/)
3. [Ember.js 入门指南之四观察者](http://blog.ddlisting.com/2016/03/17/ember-js-ru-men-zhi-nan-guan-cha-zhe-observer/)
4. [观察者官方参考文档](https://guides.emberjs.com/v2.4.0/object-model/observers/)

在下面的代码中有关计算属性部分使用的`2.0`之后的语法，在`2.0`之前计算属性的语法是不一样的（[旧语法](https://guides.emberjs.com/v1.12.0/object-model/computed-properties/)）。

修改模板`index.hbs`，把邮箱号码输入框改为Ember的`{% raw %}{{input}}{% endraw %}`助手。
```html
<!-- <input type="email" class="form-control" placeholder="Please type your e-mail address." autofocus="autofocus"/> -->
{{input type="email" value=emailAddress class="form-control" placeholder="Please type your e-mail address." autofocus="autofocus"}}
```
等待项目重启之后可以看到界面并没有变化。`{% raw %}{{input}}{% endraw %}`起到与原来代码同样的作用。<br>
值得注意的是`value=emailAddress`，并不是`value="emailAddress"`。你可以在控制器中通过名字`emailAddress`获取输入框的值。如果是`value="emailAddress"`这种方式，输入框的值默认一直都是"emailAddress"，并且在控制器中不能使用属性`emailAddress`获取值。这一点与平常我们获取`<input>`输入框的值有差别，通常获取`<input>`输入框的值是通过`name`属性获取的。修改控制器代码，在控制器中增加个计算属性和一个观察器，以及一个普通属性`emailAddress`。
```js
// app/controller/index.js

import Ember from 'ember';

export default Ember.Controller.extend({
    isDisabled: true, //设置默认值为true
    emailAddress: '',  // 设置默认值为空字符串
    // 定义一个计算属性，当属性emailAddress发生变化时会被执行不是主动执行的，是要有人调用才执行，
    // 比如执行：this.get('actualEmailAddress')去调用这个属性才会执行
    actualEmailAddress: Ember.computed('emailAddress', function() {
        console.log('actualEmailAddress function is called: ', this.get('emailAddress'));
    }),
    // 定义一个观察器，当属性emailAddress发生变化时会自动执行，也就是说观察器会检测属性emailAddress值的变化
    emailAddressChanged: Ember.observer('emailAddress', function() {
        console.log('observer is called: ', this.get('emailAddress'));
    })
});
```
下面我们做一个非常有趣的小测试。<br>
等待页面刷新完毕，打开浏览器控制台，选择标签`Ember`，在选择左侧的`/# Route`，找到`Controller`中名为`index`的，点击`$E`（如下图红色框出位置），然后再回到`Console`标签下。

![控制台](/content/images/2016/04/91.png)

点击`$E`在`Console`下可以看到类似`Ember Inspector ($E):  Class {__ember1459491972481: "ember470", __ember_meta__: Meta}`的信息。然后在控制台命令输入行输入`$E.get('actualEmailAddress')`代码的作用是获取计算属性的值。可以看到触发了计算属性方法，打印了日志，如下截图所示：

![计算属性执行日志](/content/images/2016/04/92.png)

然后再次执行`$E.get('actualEmailAddress')`计算属性方法不会被执行，因为计算属性检测的属性`emailAddress`值并没有发生变化，没有发生变化，计算属性方法不会被执行，手动修改输入框的值，结果可以看到计算属性方法再次执行了，如下图所示：

![修改输入框的值执行结果截图](/content/images/2016/04/94.png)

然后在控制台命令行在输入`$E.set('emailAddress', 'example@example.com')`这句代码意思是修改输入框的值。可以看到观察器方法执行了，因为观察器检测到被检测的属性`emailAddress`发生了变化，只要被检测的属性发生了变化就会自动执行。可以看到如下截图的日志信息：

![观察器执行结果](/content/images/2016/04/93.png)

并且可以看到邮箱号码输入框的值被置为`example@example.com`。然后在控制台命令行再次输入`$E.set('emailAddress', 'example@example.com')`观察器方法并不会执行了，即使你输入多次也不会执行，因为你输入的值`example@example.com`始终没有变化。如果你稍微修改输入的值那么可以看到观察器又执行了。比如输入`$E.set('emailAddress', 'test')`，可以看到控制台再次打印了日志信息。

测试观察器还有另外一种简单的方法，就是直接在邮件输入框直接输入某些内容。可以看到控制台会随着这输入的内容变化而变化，感觉就像是在检测键盘事件一样。下图是我输入`12@qq.com`控制台打印的日志信息：

![输入12@qq.com日志信息](/content/images/2016/04/95.png)

到此，我想你对计算属性和观察者应该有了一定的认识了！！

## 用计算属性修改isDisabled

明白了计算属性之后，用计算属性重写`isDisabled`。控制器`index.js`代码修改如下：
```js
// app/controller/index.js

import Ember from 'ember';

export default Ember.Controller.extend({
    // isDisabled: true, //设置默认值为true

    emailAddress: '',  // 设置默认值为空字符串

    isDisabled: Ember.computed('emailAddress', function() {
        return '' === this.get('emailAddress');  //判断输入框内容是否为空
    })


    // 定义一个计算属性，当属性emailAddress发生变化时会被执行不是主动执行的，是要有人调用才执行，
    // 比如执行：this.get('actualEmailAddress')去调用这个属性才会执行
    // actualEmailAddress: Ember.computed('emailAddress', function() {
    //     console.log('actualEmailAddress function is called: ', this.get('emailAddress'));
    // }),
    // 定义一个观察器，当属性emailAddress发生变化时会自动执行，也就是说观察器会检测属性emailAddress值的变化
    // emailAddressChanged: Ember.observer('emailAddress', function() {
    //     console.log('observer is called: ', this.get('emailAddress'));
    // })
});
```
直接把简单属性`isDisabled`定义为计算属性，并且这个计算属性检测`emailAddress`值的变化，如果`emailAddress`值为空那么计算属性`isDisabled`的值为`true`否则值为`false`。从而实现判断按钮“Request invitation”是否可用。Ember封装了很多字符串判断方法，直接调用Ember封装好的现成的方法，代码再修改如下：
```js
// app/controller/index.js

import Ember from 'ember';

export default Ember.Controller.extend({
    // isDisabled: true, //设置默认值为true

    emailAddress: '',  // 设置默认值为空字符串

    // isDisabled: Ember.computed('emailAddress', function() {
    //     return '' === this.get('emailAddress');  //判断输入框内容是否为空
    // })

    isDisabled: Ember.computed.empty('emailAddress')
});
```
更多有关计算属性封装好的方法请看[EMBER.COMPUTED NAMESPACE](http://emberjs.com/api/classes/Ember.computed.html)。

### isValid

记得前面“计算属性使用”这个小结提出了使用计算属性实现多个需求，其中有一个是实现判断输入的邮箱号码是否是正确格式的邮箱。现在再增加一个计算属性`isValid`判断输入的邮箱号码的格式是否正确。然后再把这个计算属性绑定到原来的计算属性`isDisabled`上。
```js
// app/controller/index.js

import Ember from 'ember';

export default Ember.Controller.extend({

    emailAddress: '',  // 设置默认值为空字符串

    emailAddress: '',  // 设置默认值为空字符串
    //  使用正则表达式判断邮箱格式，如果正确则返回true反之返回false
    isValid: Ember.computed.match('emailAddress', /^.+@.+\..+$/),
    // 把计算属性isValid绑定到isDisabled上
    isDisabled: Ember.computed.not('isValid')  //当`disabled=false`时按钮可用，所以正好需要取反
});
```
到此校验问题基本实现了，等待项目重启完成，可以看到默认状态下按钮不可用，并且当你输入的内容不符合邮箱格式时按钮也是不可用的，如果输入的内容是一个正确的邮箱那么此时按钮自动变为可用状态。不好截图，就不截图了！请读者自己试验！！

## 添加Action到控制器

目前为止，输入检验也完成了，但你输入正确邮箱后添加按钮并不会发生任何事实，输入的内容也没有保存。下面开始介绍如何处理界面输入的内容。<br>
首先修改模板`index.hbs`，在模板中增加一个`{% raw %}{{action}}{% endraw %}`标签，有关Action请看[Actions](https://guides.emberjs.com/v2.4.0/templates/actions/)。
```html
<button class="btn btn-primary btn-lg btn-block" disabled={{isDisabled}} {{action 'saveInvitation'}}>Request invitation</button>
```
仅仅修改了模板中`<button>`标签，其他不变，保存等待项目重启，此时在界面输入正确的邮箱然后点击按钮你在浏览器的控制台看到如下错误信息：

![错误信息](/content/images/2016/04/96.png)

能看到错误信息说明你的项目是正确的，因为我们并没有定义`saveInvitation`，在控制器`index`中增加这个Action的定义。
```js
// app/controller/index.js

import Ember from 'ember';

export default Ember.Controller.extend({
    // isDisabled: true, //设置默认值为true

    emailAddress: '',  // 设置默认值为空字符串

    emailAddress: '',  // 设置默认值为空字符串
    //  使用正则表达式判断邮箱格式，如果正确则返回true反之返回false
    isValid: Ember.computed.match('emailAddress', /^.+@.+\..+$/),
    // 把计算属性isValid绑定到isDisabled上
    isDisabled: Ember.computed.not('isValid'),  //当`disabled=false`时按钮可用，所以正好需要取反

    actions: {
        saveInvitation: function() {
            //  注意alert中字符串两边使用的是 `  不是单引号或者双引号
            alert(`Saving of the following email address is in propgress: ${this.get('emailAddress')}`);
            // 模拟保存操作
            this.set('responseMessage', `Thank you! We've just saved your email address: ${this.get('emailAddress')}`);
            //  情况输入框内容
            this.set('emailAddress', '');
        }
    }
});
```
**注意**：代码`alert`方法中并没有使用单引号或者是双引号囊括字符串“Saving of the following email address is in propgress: ${this.get('emailAddress')}”而是使用“\`”，这两者肯定是有区别的，前者直接把`${this.get('emailAddress')}`当着字符串，后者会把`${this.get('emailAddress')}`当着表达式，从运行结果就可以看出来了。<br>
输入正确邮箱后点击按钮会得到如下截图结果：

![结果](/content/images/2016/04/97.png)

直接弹出提示信息这种方式太暴力了，改一种提示方式，修改模板`index.hbs`，然后在注释掉控制器`index.js`中的`alert`语句。
```html
{{! app/templates/index.hbs}}

<div class="jumbotron text-center">
    <h1>Coming Soon</h1>

    <br/><br/>

    <p>Don't miss our launch date, request an invitation now.</p>

    <div class="form-horizontal form-group form-group-lg row">
        <div class="col-xs-10 col-xs-offset-1 col-sm-6 col-sm-offset-1 col-md-5 col-md-offset-2">
          <!-- <input type="email" class="form-control" placeholder="Please type your e-mail address." autofocus="autofocus"/> -->
          {{input type="email" value=emailAddress class="form-control" placeholder="Please type your e-mail address." autofocus="autofocus"}}
        </div>
        <div class="col-xs-10 col-xs-offset-1 col-sm-offset-0 col-sm-4 col-md-3">
            <button class="btn btn-primary btn-lg btn-block" disabled={{isDisabled}} {{action 'saveInvitation'}}>Request invitation</button>
        </div>

    </div>

    {{! 显示提示信息}}
    {{#if responseMessage}}
     <div class="alert alert-success">{{responseMessage}}</div>
   {{/if}}


   <br/><br/>
</div>
```
上述代码新引入了一个表达式`{% raw %}{{if}}{% endraw %}`，顾名思义，这个表达式就是用于判断的。更多有关判断表达式的介绍请看[Ember.js 入门指南之九handlebars条件表达式](http://blog.ddlisting.com/2016/03/18/ember-js-ru-men-zhi-nan-zhi-jiu-handlebarstiao-jian-biao-da-shi/)

等待项目重启完成，再次测试。输入正确格式的邮箱，点击按钮提交可以看到如下的结果：

![友好提示信息](/content/images/2016/04/98.png)

主要看绿色提示信息，相对于前一种直接弹框提示方式友好多了！！！

好了，到此第二篇也完成了。内容比较多需要耐心才能看完，如果你能认真坚持看到这里相信你一定收获了很多！！


## 家庭作业

**作业要求**

1. 一个邮件输入框`{% raw %}{{input}}{% endraw %}`，需要校验不为空、校验邮箱格式
2. 一个消息输入框`{% raw %}{{textarea}}{% endraw %}`，需要校验不为空、输入信息长度不少于5
3. 上述两个输入框的校验都通过才允许提交
4. 提交成功后在界面显示提示信息

**用到的组件和方法**

```js
{{input type="email" value=emailAddress class="form-control" placeholder="Please type your e-mail address." autofocus="autofocus"}}
```

```js
{{textarea class="form-control" placeholder="Your message. (At least 5 characters.)" rows="7" value=message}}
```

判断长度不小于5用到的方法。
```js
Ember.computed.gte('propertyName', number);
```

并且判断。
```js
Ember.computed.and('firstProperty', 'secondProperty');
```

获取属性值的长度。
```js
message.length
```

获取计算属性值长度
```js
Ember.computed('propertyName', function() {
    return this.get('propertyName').get('length');
});
```


<br>
为了照顾懒人我把完整的代码放在[GitHub](https://github.com/ubuntuvim/library-app)上，如有需要请参考参考。博文经过多次修改，博文上的代码与github代码可能有出入，不过影响不大！如果你觉得博文对你有点用，请在github项目上给我点个`star`吧。您的肯定对我来说是最大的动力！！
