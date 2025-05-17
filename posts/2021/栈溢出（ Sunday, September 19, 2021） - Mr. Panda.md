---
date: 2022-01-01
title: 栈溢出（ Sunday, September 19, 2021） - Mr. Panda
# category: 
# tags: 
# description:
---

# 栈溢出（ Sunday, September 19, 2021） - Mr. Panda

---
栈溢出系列文章——解决技术问题，积累开发经验。

本期内容包括：React: 怎么修复 useRef “Cannot Assign to … Read Only Property” 的错误？、TypeScript：ts-node直接运行typescript文件等。

## React: 怎么修复 useRef “Cannot Assign to ... Read Only Property” 的错误？

如果你遇到了这种错误，很有可能是 useRef 的值的类型被定义成了 React.RefObject，因为 React.RefObject 会将 current 引用定义为 readonly。

```typescript
interface RefObject<T> {
  readonly current: T | null;
}
```

```typescript
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}
```

1.  useRef 的初始值为null；
2.  cuurent 属性的类型被初始化为一个确定的类型。

例如：

```typescript
const elem = useRef<HTMLDivElement>(null);
type Test = typeof elem; // Test = React.RefObject<HTMLDivElement>
```

```typescript
const elem = useRef<HTMLDivElement | null>(null);
type Test = typeof elem; // Test = React.MutableRefObject<HTMLDivElement|null>
```

正常 ts 文件都要编译成 js 文件才能运行，但是在开发时有时需要运行 ts 文件，可通过 ts-node 直接运行 ts 文件。

```bash
//全局安装typescript和ts-node
npm install -g typescript
//npm install -g typescript-node 由于typescript-node不支持更高版本的ts
npm install -g ts-node //typescript@>=2.7

ts-node foo.ts
```

-   (…args: never\[\]) => void: 意思是函数不接受任何参数。
-   (…args: any\[\]) => void：意思是函数可接受任意参数。

参见：

-   [TypeScript type signatures for functions with variable argument counts](https://stackoverflow.com/questions/12739149/typescript-type-signatures-for-functions-with-variable-argument-counts)；
-   [Understanding `(...args: never[]) => ...` in TypeScript Handbook](https://www.reddit.com/r/typescript/comments/muyl55/understanding_args_never_in_typescript_handbook/)；

## Mac: 解决 homebrew 下载过慢问题

使用国内镜像源替换homebrew镜像源。把三个仓库地址全部替换成国内Alibaba提供的地址：

-   替换/还原brew.git仓库地址

```bash
# 替换成阿里巴巴的 brew.git 仓库地址:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
 
#=======================================================
 
# 还原为官方提供的 brew.git 仓库地址
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
```

```bash
替换成阿里巴巴的 homebrew-core.git 仓库地址:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
=======================================================
还原为官方提供的 homebrew-core.git 仓库地址
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```

这个步骤跟你的macOs系统使用的shell版本有关系，首先查看shell版本

```bash
echo $SHELL
```

-   zsh终端操作方式

```bash
# 替换成阿里巴巴的 homebrew-bottles 访问地址:
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
 
#=======================================================
 
# 还原为官方提供的 homebrew-bottles 访问地址
vi ~/.zshrc
# 然后，删除 HOMEBREW_BOTTLE_DOMAIN 这一行配置
source ~/.zshrc
```

```bash
# 替换 homebrew-bottles 访问 URL: 
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
 
#=======================================================
 
# 还原为官方提供的 homebrew-bottles 访问地址
vi ~/.bash_profile
# 然后，删除 HOMEBREW_BOTTLE_DOMAIN 这一行配置
source ~/.bash_profile
```

### call

-   功能：执行 function、明确指定 this 指向。
-   语法：fn.call(this, arg1, arg2..., argn)。
-   使用情境：需要明确指定 this 的指向。

### apply

apply 和 call 功能和场景使用一样，不同的是 apply 可以以数组的方式传参。

语法：fn.apply(this, \[arg1, arg2..., argn\])

### bind

-   功能：明确指定 this 指向、返回一个函数（注意：bind 不会执行此函数）。
-   语法：fn.bind(this, arg1, arg2..., argn)。
-   bind 除了可以明确指定 this 指向之外，还可以预先传入一些参数给目标方法（类似于 Currying Function）。当我们需要先记住部分参数或者对不断传入固定参数给方法是，可以使用 bind 起到简化语法的作用。
-   bind polyfill

```javascript
function bind(t, callback) {
  var outerArgs = Array.from(arguments).slice(2);
  return function () {
    var innerArgs = Array.from(arguments);
    return callback.apply(t, outerArgs.concat(innerArgs));
  };
}

function add() {
  return Array.from(arguments).reduce(function (sum, num) {
    return sum + num;
  });
}

var addWithBind = bind(null, add, 1, 5);
console.log(addWithBind(8)); // 14
```

### 链接格式

```markup
<a href="[scheme]://[host]/[path]?[query]"> 唤起应用 </a>
```

### 常见 URL Scheme

-   系统默认应用

| 名称      | URL Scheme                   | Bundle identifier |
| --------- | ---------------------------- | ----------------- |
| Safari    | http://                      |                   |
| maps      | http://maps.google.com       |                   |
| Phone     | tel://                       |                   |
| SMS       | sms://                       |                   |
| Mail      | mailto://                    |                   |
| iBooks    | ibooks://                    |                   |
| App Store | itms-apps://itunes.apple.com |                   |
| Music     | music://                     |                   |
| Videos    | videos://                    |                   |

系统默认应用常见 URL Scheme

-   第三方软件

| 名称             | URL Scheme         | Bundle identifier                  |
| ---------------- | ------------------ | ---------------------------------- |
| QQ               | mqq://             |                                    |
| 微信             | weixin://          |                                    |
| 腾讯微博         | TencentWeibo://    |                                    |
| 淘宝             | taobao://          |                                    |
| 支付宝           | alipay://          |                                    |
| 微博             | sinaweibo://       |                                    |
| weico微博        | weico://           |                                    |
| QQ浏览器         | mqqbrowser://      | com.tencent.mttlite                |
| uc浏览器         | dolphin://         | com.dolphin.browser.iphone.chinese |
| 欧朋浏览器       | ohttp://           | com.oupeng.mini                    |
| 搜狗浏览器       | SogouMSE://        | com.sogou.SogouExplorerMobile      |
| 百度地图         | baidumap://        | com.baidu.map                      |
| Chrome           | googlechrome://    |                                    |
| 优酷             | youku://           |                                    |
| 京东             | openapp.jdmoble:// |                                    |
| 人人             | renren://          |                                    |
| 美团             | imeituan://        |                                    |
| 1号店            | wccbyihaodian://   |                                    |
| 我查查           | wcc://             |                                    |
| 有道词典         | yddictproapp://    |                                    |
| 知乎             | zhihu://           |                                    |
| 点评             | dianping://        |                                    |
| 微盘             | sinavdisk://       |                                    |
| 豆瓣fm           | doubanradio://     |                                    |
| 网易公开课       | ntesopen://        |                                    |
| 名片全能王       | camcard://         |                                    |
| QQ音乐           | qqmusic://         |                                    |
| 腾讯视频         | tenvideo://        |                                    |
| 豆瓣电影         | doubanmovie://     |                                    |
| 网易云音乐       | orpheus://         |                                    |
| 网易新闻         | newsapp://         |                                    |
| 网易应用         | apper://           |                                    |
| 网易彩票         | ntescaipiao://     |                                    |
| 有道云笔记       | youdaonote://      |                                    |
| 多看             | duokan-reader://   |                                    |
| 全国空气质量指数 | dirtybeijing://    |                                    |
| 百度音乐         | baidumusic://      |                                    |
| 下厨房           | xcfapp://          |                                    |

第三方软件常见 URL Scheme

## VSCode: 添加自定义代码段

参见：[\[VS Code\]跟我一起在Visual Studio Code 添加自定义snippet（代码段），附详细配置](https://blog.csdn.net/maokelong95/article/details/54379046)；

## Mac: 自定义快捷键打开应用程序

参见：[在Mac OS X 自定义快捷键打开应用](https://www.jianshu.com/p/aa69812b2303)；

## Mac: 使用终端打开应用程序

### **打开任意路径下的一个应用程序**

打开命令通常需要你在当前路径中输入完整的文件路径。但是，在打开命令的后面加上-a，会让终端打开这个应用程序。

```bash
# 如果想要打开iTunes程序：
open -a iTunes
# 如果应用程序的名字中存在空格，请使用引号：
open -a "App Store"
```

输入文件路径和文件名，后面加上-a和应用程序的名字。

```bash
# 使用文本编辑器来打开一个.doc文件：
open Downloads/Instructions.doc -a TextEdit
```

输入**info open**来查看修改打开命令的选项列表，info 是 Linux 命令，作用是查看命令帮助信息。

```bash
# 使用-e来指定“文本编辑器”，-t代表默认的文本程序：
open Downloads/Instructions.doc -e
# 加上-g选项会让应用程序留在后台，这样你就会继续停留在终端程序中：
open -g -a iTunes
```

## VSCode: ctrl+s 后会在当前目录下自动生成dist目录

解决办法：关闭`compile-hero`插件。

## Git: Git 撤销和回滚操作

参见：[Git撤销&回滚操作](https://blog.csdn.net/ligang2585116/article/details/71094887)；

## Yarn: yarn 安装 node-sass 速度缓慢

```bash
# 配置 node-sass 的二进制包镜像地址为淘宝源
yarn(npm) config set sass_binary_site http://cdn.npm.taobao.org/dist/node-sass -g
```

在项目中添加 _declarations.d.ts_ 文件：

```typescript
// We need to tell TypeScript that when we write "import styles from './styles.scss' we mean to load a module (to look for a './styles.scss.d.ts'). 
declare module '*.scss'; 
```

## Git: 更换远程仓库地址

```bash
方法一 ： 通过命令直接修改远程仓库地址
git remote 查看所有远程仓库
git remote xxx 查看指定远程仓库地址
git remote set-url origin 你新的远程仓库地址

方法二： 先删除在添加你的远程仓库
git remote rm origin
git remote add origin 你的新远程仓库地址
```

```bash
yarn install --update-checksums
```

1.  打开 Keychain Access.app；
2.  搜索 wifi 名称；
3.  右键：将密码拷贝到剪贴板。

## CSS：隐藏滚动条并可以滚动内容

```css
// webkit .element::-webkit-scrollbar { width: 0 !important } // IE 10+ .element { -ms-overflow-style: none; } // Firefox .element { overflow: -moz-scrollbars-none; }
```

## CSS: 数字字母自动换行

汉字会自动换行，但数字和字母不会换行的解决办法：

word-break:break-all或word-wrap:break-word。

## CSS: 高度随宽度比例变化

### padding-bottom 实现

-   padding 设置为百分比时是相对于父元素宽度而言的。
-   可以使用百分比的 padding-bottom 代替 height 来实现高度与宽度成比例的效果。
-   将 padding-bottom 设置为想要实现的 height 的值，同时将 height 设置为 0 以使元素的“高度”等于 padding-bottom 的值。

### 隐藏的图片

-   div 容器如果不给定高度，它的高度会随着容器内部的元素变化而变化。
-   在容器内部添加一张符合宽高比例的图片，给图片设置宽度100%，高度auto。
-   图片只设置宽度的话，高度会随宽度来进行比例变化，这样内部的子容器的高度也会按照比例缩放。
-   图片占位隐藏，也可以用别的盒子覆盖上。
-   这种方法不需要考虑任何兼容性。由于图片只需要一个形状，可以使用 base64图片来减少 http 请求。

### vw 和 vh

将父容器的宽度和高度定义为相同的vw，这样父容器的高度和宽度就是相同值，子容器的宽高值设为百分比，不管父容器大小如何变，子容器的高度和宽s度比都是不会变的。

-   vw： 相对于视窗的宽度
-   vh： 相对于视窗的高度
-   vmin： 相对于视口的宽度或高度中较小的
-   vmax：相对于视口的宽度或高度中较大的

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
