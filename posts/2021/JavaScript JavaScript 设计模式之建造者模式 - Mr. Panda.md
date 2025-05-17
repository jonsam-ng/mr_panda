---
date: 2022-01-01
title: JavaScript 设计模式之建造者模式 - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] JavaScript 设计模式之建造者模式 - Mr. Panda

---
## 简介

建造者模式注重创建对象之细节，创建出的**复杂对象或者复合对象**结构清晰。相比于工厂模式，工厂模式更注重于创建与管理类，建造者模式注重于创建更为复杂的类。我们可以将工厂模式和建造者模式结合使用。我们使用建造者模式的目的是将复杂的对象进行拆分，方便我们用面向对象的语言对业务进行描述，减小对象的复杂性，使之建构简单清晰、容易维护。

## 使用场景

-   需要创建的类比较复杂，需要使用建造者模式进行拆分时。
-   对复杂的类进行拆分，可以将复杂的逻辑简单化。

## 为什么使用

-   方便创建复杂对象或者复合对象，保持清晰的代码结构。
-   将复杂的逻辑进行解耦。

## 如何使用

-   将复杂对象拆成若干简单对象的组合。

## 代码案例

```javascript
建造者模式实例

class Wordpress{
 //    初始化 wordpress 配置
     constructor(plugins, themes, posts) {
         this.plugins = plugins;
         this.themes = themes;
         this.posts = posts;
         this.currentTheme = this.themes[0]; 
     }
 //    安装插件
     installPlugin(plugin) {
         this.plugins.push(plugin);
     }
 //    切换主题
     switchTheme(name) {
         this.currentTheme = this.themes.filter((theme) => theme.name === name);
     }
 //    写文章
     writePost(post) {
         this.posts.push(post);
     }
 }
 class Plugin{
     constructor(name, detail) {
         this.name = name;
         this.detail = detail;
     }
 }
 class Theme{
     constructor(name, author) {
         this.name = name;
         this.author = author;
     }
 }
 class Post{
     constructor(title, author, content) {
         this.title = title;
         this.author = author;
         this.content = content;
     }
 }
 // 从数据库获取的结构化数据
 const wp = {
     plugins: [
         {
             name: 'SEO工具',
             detail: '帮助您优化网站 SEO',  
         },
         {
             name: 'Wp-Template',
             detail: '帮助您创建自定义文章模板', 
         }
     ],
     themes: [
         {
             name: 'Twentieth Theme',
             author: 'jonsam',
         },
         {
             name: 'Aborsh Gloah',
             author: 'john alark',
         }
     ],
     posts: [
         {
             name: '如何让您的网站更好的被搜索？',
             author: 'jonsam',
             content: '第一篇文章'
         },
         {
             name: 'Vue 的更新会有什么影响',
             author: 'jonsam',
             content: '第二篇文章'
         },
 ]
 }
 // 实例化插件、主题、文章和 WordPress。
 const plugins = wp.plugins.map(plugin => new Plugin(plugin.name, plugin.detail));
 const themes = wp.themes.map(theme => new Theme(theme.name, theme.author));
 const posts = wp.posts.map(post => new Post(post.title, post.author, post.content));
 const wordpress = new Wordpress(plugins, themes, posts);
 wordpress.installPlugin(new Plugin('Aka Search', '一个美观的搜索插件'));
 wordpress.switchTheme('Aborsh Gloah');
 wordpress.writePost(new Post('迷失在黑暗中的梅花鹿', 'jonsam', '这是一篇感想'));
 console.log(wordpress);

>>>

Wordpress {
   plugins:
    [ Plugin { name: 'SEO工具', detail: '帮助您优化网站 SEO' },
      Plugin { name: 'Wp-Template', detail: '帮助您创建自定义文章模板' },
      Plugin { name: 'Aka Search', detail: '一个美观的搜索插件' } ],
   themes:
    [ Theme { name: 'Twentieth Theme', author: 'jonsam' },
      Theme { name: 'Aborsh Gloah', author: 'john alark' } ],
   posts:
    [ Post { title: undefined, author: 'jonsam', content: '第一篇文章' },
      Post { title: undefined, author: 'jonsam', content: '第二篇文章' },
      Post { title: '迷失在黑暗中的梅花鹿', author: 'jonsam', content: '这是一篇感想' } ],
   currentTheme: [ Theme { name: 'Aborsh Gloah', author: 'john alark' } ] }
```
