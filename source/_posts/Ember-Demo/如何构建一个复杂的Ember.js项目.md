---
title: 如何构建一个复杂的Ember.js项目
tag:
  - Emberjs
  - Ember-Demo 
---

本系列教材将为读者介绍怎么样使用Ember.js构建一个复杂的项目。本教程分为6个小部分，通过这6篇文章一步步为你讲解怎么使用Ember.js构建一个稍微复杂的Ember.js项目。

有关Ember.js的前世今生我就不多做介绍了，请自行查看官方[参考文档](http://emberjs.com)

提醒：如果可以最好是在看一遍官方参考文档之后再看本系列教程，有助于把你所学的零碎的有关Ember的知识串联起来，否则，可能你会看得比较痛苦，建议是先把这6篇文章认真看一遍下来再自己动手，按照文章提供的源码自己再实践一遍。

**目录**

1. [环境搭建以及使用Ember.js创建第一个静态页面](http://xcoding.tech/2016/03/30/Ember-Demo/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E4%BB%A5%E5%8F%8A%E4%BD%BF%E7%94%A8Ember.js%E5%88%9B%E5%BB%BA%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%9D%99%E6%80%81%E9%A1%B5%E9%9D%A2/)
2. [引入计算属性、action、动态内容](http://xcoding.tech/2016/03/31/Ember-Demo/%E5%BC%95%E5%85%A5%E8%AE%A1%E7%AE%97%E5%B1%9E%E6%80%A7%E3%80%81action%E3%80%81%E5%8A%A8%E6%80%81%E5%86%85%E5%AE%B9/)
3. [模型，保存数据到数据库](http://xcoding.tech/2016/03/31/Ember-Demo/%E6%A8%A1%E5%9E%8B%EF%BC%8C%E4%BF%9D%E5%AD%98%E6%95%B0%E6%8D%AE%E5%88%B0%E6%95%B0%E6%8D%AE%E5%BA%93/)
4. [发布项目，加入CRUD功能](http://xcoding.tech/2016/03/31/Ember-Demo/%E5%8F%91%E5%B8%83%E9%A1%B9%E7%9B%AE%EF%BC%8C%E5%8A%A0%E5%85%A5CRUD%E5%8A%9F%E8%83%BD/)
5. [从服务器获取数据，引入组件](http://xcoding.tech/2016/03/31/Ember-Demo/%E4%BB%8E%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%8E%B7%E5%8F%96%E6%95%B0%E6%8D%AE%EF%BC%8C%E5%BC%95%E5%85%A5%E7%BB%84%E4%BB%B6/)
6. [模型高级特性，引入模型关联关系](http://xcoding.tech/2016/03/31/Ember-Demo/%E6%A8%A1%E5%9E%8B%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%EF%BC%8C%E5%BC%95%E5%85%A5%E6%A8%A1%E5%9E%8B%E5%85%B3%E8%81%94%E5%85%B3%E7%B3%BB/)


**项目软件环境**

1. [NodeJS 5.9.1](https://nodejs.org/en/)
2. [Ember CLI 2.4.3](http://www.ember-cli.com/user-guide/)
3. [chrome插件Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en)，如果无法访问那你应该要去fanqiang了！！
4. [Watchman（可选，如果是Mac系统推荐安装）](https://facebook.github.io/watchman/)

上述软件请自行安装提供的网址安装，如果安装不成功，或者安装出现错误，请谷歌、百度。如果还是解决不了给我留言获取是去[Ember 社区](http://discuss.emberjs.com/)提问。


**说明：本教程是基于Ember2.4而作，请注意与你自己的Ember.js版本区别，如果出现不兼容问题请自行升级项目。**

升级教程：[http://blog.ddlisting.com/2015/11/24/sheng-ji-emberdao-2-2-0ban-ben/](http://blog.ddlisting.com/2015/11/24/sheng-ji-emberdao-2-2-0ban-ben/)
本教程是介绍升级到2.2版本的，不过同样的道理，只需要修改对应的版本为2.4即可。
