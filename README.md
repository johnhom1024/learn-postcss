# PostCss Learning

## 前言

这个项目是用来展示PostCss的一个简单使用和作用的项目。

接下来我们就通过Webpack + PostCss + autoprefixer这三个工具来演示，如何给css属性自动添加前缀，以支持各个浏览器厂商们的标准。这个项目需要一点Webpack的基础

## 什么是PostCss

PostCSS 是一个使用 JS 插件转换样式的工具。

PostCSS获取一个CSS文件，并提供一个API来分析和修改其规则（通过将其转换为抽象语法树）。这个API可以被插件用来做很多有用的事情，例如，自动查找错误，或者插入供应商前缀。


## 浏览器样式前缀

> 浏览器厂商们有时会给实验性的或者非标准的 CSS 属性和 JavaScript API 添加前缀，这样开发者就可以用这些新的特性进行试验，同时（理论上）防止他们的试验代码被依赖，从而在标准化过程中破坏 web 开发者的代码。开发者应该等到浏览器行为标准化之后再使用未加前缀的属性。

### CSS前缀

主流的浏览器引擎前缀：

* `-webkit-` 基于WebKit内核的浏览器。
* `-moz-` 火狐浏览器。
* `-o-` 旧版Opera浏览器。
* `-ms-` IE浏览器和Edge浏览器。

示例：

```css
-webkit-transition: all 4s ease;
-moz-transition: all 4s ease;
-ms-transition: all 4s ease;
-o-transition: all 4s ease;
transition: all 4s ease; 
```

通过使用`autoprefixer`可以根据项目中的css属性，自动生成对应浏览器的前缀。

## PostCss实践

通过以上的两节，我们对PostCss和autoprefixer这两个工具有了一个基本的了解。下面我们在Webpack上引入这两个工具，对js中引入的css文件进行一个转换，自动加上的浏览器前缀，然后生成一个新的css文件。

构建一个js项目：

```
mkdir postcss-learn
npm init
npm install -D css-loader style-loader webpack webpack-cli mini-css-extract-plugin

# 安装PostCss相关依赖
npm install -D postcss postcss-loader
```

我们使用postcss.config.js文件来对PostCss做一些相关的配置，现在我们加入autoprefixer。

安装autoprefixer:

```
npm install -D autoprefixer
```

新增postcss.config.js文件，内容如下：

```js
// postcss.config.js
module.exports = {
  plugins: [
    require("autoprefixer")
  ],
};
```

autoprefixer同样也需要配置文件，我们在根目录下新增`.browserslistrc`文件，内容如下：

```
> 1%
last 2 versions
IE 10
not ie <= 8
```

> browserslistrc相关配置可以在https://github.com/browserslist/browserslist中查看

接下来我们配置webpack。

项目根目录下新增webpack.config.js文件，内容如下：

```js
// ./webpack.config.js
const path = require("path");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  mode: "development",
  // 将要转换的js文件入口
  entry: "./src/js/index.js",
  output: {
    // 输出的js文件名
    filename: "bundle.js",
    // 输出的文件夹地址
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "postcss-loader"],
      },
    ],
  },
  plugins: [
    new MiniCssExtractPlugin({
      // 生成的css文件名
      filename: "main.css",
    }),
  ],
};
```

接下来我们简单写一下js和css:

```js
// .src/js/index.js

import '../style/index.css'

const div = document.createElement('div')
div.innerHTML = 'hello postcss'
div.className = 'hello_postcss'
document.body.append(div)
```

```js
// ./src/style/index.css

.hello_postcss {
  color: #00dd00;
  display: flex;
}
```

最后一步我们用Webpack打包：

```
npx webpack
```

运行成功之后，Webpack会在项目根目录位置生成一个dist文件夹，里面有一个bundle.js和一个main.css文件，我们可以查看main.css的内容：

```
.hello_postcss {
  color: #00dd00;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
}
```

我们可以看到flex的属性值已经加上了支持webkit和IE/Edge的浏览器前缀。

## 总结

index.js引入了index.css文件之后，会被webpack中的rule规则识别，然后按照从右到左的顺序交给postloader、css-loader、MiniCssExtractPlugin.loader进行处理。

以上就是一个简单的PostCss + autoprefixer的演示项目。