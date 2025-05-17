---
date: 2022-01-01
title: 如何在 React 中使用 SVG - Mr. Panda
# category: 
# tags: 
# description:
---

# [React] 如何在 React 中使用 SVG - Mr. Panda

---
## 什么是 SVG？

SVG 是一种基于[XML](https://en.wikipedia.org/wiki/XML)的矢量图形图像格式。SVG 代表可缩放矢量图形。它是在 1990 年代后期开发的，直到 2016 年左右[才得到很好的支持](https://www.sitepoint.com/svg-101-what-is-svg/)。今天，很大比例的图标库，如 Flaticon、Font Awesome、Material Icon 等，都完全支持 SVG。Twitter、Youtube、Udacity、Netflix 等品牌在他们的一些图像和图标中使用了 SVG。

## 为什么 SVG 优于其他图像格式？

使用 SVG 相对于其他图像格式（JPEG、GIF、PNG）的一些优势：

-   **可扩展性和分辨率**：这是 SVG 的最大优势，SVG 使用形状、数字和坐标，而不是像其他图像格式那样使用像素网格。这使得在不损失任何质量的情况下放大和缩小 SVG 图像成为可能，并使 SVG 能够无限缩放。
-   **最小文件大小**：与其他文件格式相比，SVG 图像的文件大小通常较小，并且易于压缩，因此有可能变得更小。
-   **高性能和速度**：由于 SVG 图像的小尺寸特性，浏览器渲染它们变得非常容易和快速。与为其他图像格式渲染像素和颜色相比，它就像渲染文本。此外，如果您在代码中使用**内联 SVG**，浏览器不必发出请求来获取图像并像文件中的所有其他代码一样呈现它。在这种情况下，没有发出请求。但是在您拥有复杂的图像 SVG 文件的情况下，我建议使用 PNG 或 JPEG，因为 SVG 的加载时间和性能会急剧下降。
-   **DOM-like and style-able** : SVG 图像就像代码一样，所以这意味着它可以像 DOM 元素一样导航并且也可以设置样式，尽管某些属性有不同的名称，也可以用 CSS 设置样式。
-   **可编辑**：由于 SVG 类似于 DOM，因此可以使用任何文本编辑器创建、编辑和动画化 SVG。
-   **集成**：SVG 可以以多种方式使用，可用于显示徽标图像和图标。它可以用作图形、动画和效果。
-   **Animatable** : SVG 可以被动画化。这可以使用 **Web Animation API、WebGL、CSS 动画**等工具来完成。
-   **可访问性和 SEO**：SVG 包含提高网站可访问性的文本，这也意味着它们可以被搜索、索引、编写脚本等。

## 如何在 React 中使用 SVG

下面我们将介绍可以在网页上使用或呈现此 React SVG 徽标的各种方式。

### **为静态 SVG 使用图像标签**

为了能够在 img 中使用 SVG 或任何其他图像格式，我们必须在我们使用的任何模块打包器（Webpack、Parcel 等）中设置文件loader。如果您已经在使用 Webpack，我将在这里向您展示如何通过几个步骤进行设置。

首先，我们安装文件加载器库`$ npm install file-loader --save-dev`，这会将其安装为开发依赖项。

您可以使用以下代码更新您的 Webpack 配置文件规则：

```javascript
const webpack = require("webpack"); module.exports = { entry: "./src/index.js", module: { rules: [ //... { test: /\.(png|jp(e*)g|svg|gif)$/, use: [ { loader: "file-loader", options: { name: "images/[hash]-[name].[ext]", }, }, ], }, ], }, //... };
```

### **使用 SVG 标签**

使用上面相同的 Webpack 设置，我们可以使用 SVG 标签，用法是将`.svg`文件的内容复制并粘贴到您的代码中。这是一个示例：

```javascript
import React from "react"; import ReactLogo from "./logo.svg"; const App = () => { return ( <div className="App"> <img src={ReactLogo} alt="React Logo" /> </div> ); }; export default App;
```

使用这种方法的缺点是，当图像更复杂时，SVG 文件会变得更大，并且由于 SVG 存储在文本中，这意味着我们的代码中有一大堆文本。

### **使用 SVG 作为组件**

SVG 可以作为 React 代码中的 React 组件直接导入和使用。图像不会作为单独的文件加载，而是沿 HTML 呈现。示例用例如下所示：

```javascript
import React from "react"; import { ReactComponent as ReactLogo } from "./logo.svg"; const App = () => { return ( <div className="App"> <ReactLogo /> </div> ); }; export default App;
```

请注意，此方法仅适用于 create-react-app，如果您不使用 create-react-app，我建议您使用其他方法。Create-react-app 在后台使用 SVGR 可以将 SVG 转换和导入为 React 组件。

### **使用 SVGR**

[SVGR](https://react-svgr.com/)是一个很棒的工具，可以将你的 SVG 转换成你可以使用的 React 组件。那么我们该如何设置呢？

首先，我们安装包`$ npm install @svgr/webpack --save-dev`。

然后我们更新我们的 Webpack 配置规则以将 SVGR 用于 SVG：

```javascript
const webpack = require("webpack"); module.exports = { entry: "./src/index.js", module: { rules: [ //... { test: /\.svg$/, use: ["@svgr/webpack"], }, ], }, //... };
```

现在我们可以将 SVG 图像作为 React 组件导入并在我们的代码中使用它，如下所示：

```javascript
import React from "react"; import ReactLogo from "./logo.svg"; const App = () => { return ( <div className="App"> <ReactLogo /> </div> ); }; export default App;
```

### 使用 SVG 作为 data-url

Data-url 是以`data:`作为前缀允许内容创建者在文档中嵌入小文件的方案。这种方法允许我们像使用内联元素一样使用 SVG 图像。

首先，我们需要一个合适的 Webpack 加载器，在用例中，我将使用 svg-url-loader。我们可以通过运行这个命令将它安装到我们的项目中 `npm install svg-url-loader --save-dev`。然后我们将使用这些规则更新 Webpack 配置文件的规则部分：

```javascript
const webpack = require("webpack"); module.exports = { entry: "./src/index.js", module: { rules: [ //... { test: /\.svg$/, use: [ { loader: "svg-url-loader", options: { limit: 10000, }, }, ], }, ], }, //... };
```

现在我们可以导入我们的 SVG 文件并在我们的 React 组件中使用它，如下所示：

```javascript
import ReactLogo from './logo.svg'; const App = () => { return ( <div className="App"> <img src={ReactLogo} alt="React Logo" /> </div> ); } <img src="ebbc8779488b4073081931bd519478e8.svg" alt="" />
```

## 注意事项

-   **复杂**的图像：图像越复杂，SVG 文件就越大，就像我们在尝试使用 SVG 标签时看到的那样。在这种情况下，建议您使用 PNG 或 JPEG。
-   兼容性：SVG 没有向后浏览器支持，这意味着并非所有旧版本的浏览器都支持 SVG，因此 SVG 可能无法在这些浏览器中正确显示。这是因为最初引入 SVG 时，大多数浏览器都不支持 SVG。如果您的目标是旧版本的浏览器，SVG 可能不是您的首选图像格式。

## 结论

SVG 对于构建快速、高性能和可访问的网页非常有用。我建议您将 SVG 用于低强度图像，例如徽标、图标和不太复杂的图像。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
