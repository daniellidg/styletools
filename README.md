做前端开发，你一定或多或少的用过一些CSS工具库，比如[Bootstrap](https://getbootstrap.com/), [Tachyons](https://tachyons.io/)等。

但是不同的库对于类名的定义不尽相同，使用起来难免有些不太得心应手。

本文将教你怎么用sass一步一步构建自己的样式库。

其实不难。如果对sass用法还不太熟悉，可以先看下这篇文章[SASS用法指南](https://www.ruanyifeng.com/blog/2012/06/sass.html)。

### 环境配置
首先需要安装[node-sass](https://www.npmjs.com/package/node-sass)，后续我们会用它将.scss文件编译成.css文件
```
npm install -g node-sass
```
接下来我们新建一个文件style.scss

### 定义网页基础样式
定义一些共通的基础样式：
```scss
* {
  box-sizing: border-box;
  outline: none;
}

html {
  font-family: sans-serif;
  line-height: 1.15;
  -webkit-text-size-adjust: 100%;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
}

article, aside, figcaption, figure, footer, header, hgroup, main, nav, section {
  display: block;
}

body {
  margin: 0;
  font-family: "Apple Color Emoji", "Segoe UI Emoji", sans-serif;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: #212529;
  text-align: left;
  background-color: #fff;
}

h1, h2, h3, h4, h5, h6 {
  margin-top: 0;
  margin-bottom: 0.5rem;
}

......
```

这部分不做详细的介绍，可以根据自己的需要随意的增减。

#### 构建字体尺寸样式
下面以构建字体尺寸为例，介绍如何只用几行代码就可以一次构建多个字体尺寸类。

首先新建一个字体尺寸的Map类型数据结构。
```scss
$fontSizeMap: (
  0: 0,
  1: 0.5rem,
  2: 1rem,
  3: 1.5rem,
  4: 3rem,
);
```
然后通过@each对$fontSizeMap进行遍历：
```scss
@each $key, $val in $fontSizeMap {
  .ft-#{$key} {
    font-size: #{$val};
  }
}
```

在终端编译一下style.scss
```
  node-sass style.scss style.css
```
如果编译成功，会在同级目录下生成一个style.css文件。

打开style.css文件，我们通过scss创建的字体尺寸的类就已经自动编译好了。
```css
.ft-0 {
  font-size: 0; }

.ft-1 {
  font-size: 0.5rem; }

.ft-2 {
  font-size: 1rem; }

.ft-3 {
  font-size: 1.5rem; }

.ft-4 {
  font-size: 3rem; }
```

以后如果需要增加新的尺寸的字体，直接在$fontSizeMap中添加一条数据，然后重新编译一下就好了。

使用同样的方式，我们可以很方便的构建一整套其他样式，比如：字重，文字颜色，背景颜色，边框大小等等。

### 构建margin/padding样式
接下来介绍一个复杂一点的，以margin/padding为例。

最终要生成下面这三类样式：
  1. 每条边：类似mt0, mr0, mb0, ml0
  2. 对角边: mx0, my0
  3. 所有边: ma0

首先要定义三个Map类型数据结构

1. 类型:
```scss
  $spaceTypes: (
    m: margin,
    p: padding
  );
```

2. 方向:
```scss
$spaceDirections: (
  t: top,
  r: right,
  b: bottom,
  l: left
);
```

3. 边距大小
```scss
$spaceSizes: (
  0: 0,
  1: .5rem,
  2: 1rem,
  3: 1.5rem,
  4: 3rem,
);
```

然后同样利用@each函数遍历以上三个数据结构，但是需要多层嵌套:
```scss
@each $typeKey, $type in $spaceTypes {
  @each $sizeKey, $size in $spaceSizes {
    // mt0, mr0, mb0, ml0
    @each $directionKey, $direction in $spaceDirections {
      .#{$typeKey}#{$directionKey}#{$sizeKey} {
        #{$type}-#{$direction}: $size;
      }
    }

    // mx0, my0
    .#{$typeKey}x#{$sizeKey} {
      #{$type}-left: $size;
      #{$type}-right: $size;
    }
    .#{$typeKey}y#{$sizeKey} {
      #{$type}-top: $size;
      #{$type}-bottom: $size;
    }

    // ma0
    .#{$typeKey}a#{$sizeKey} {
      #{$type}: $size;
    }
  }
}
```

再次编译style.scss，就能看到我们一次创建了70个类。
```scss
$spaceTypes.length * ($spaceDirections.length + 3) * $spaceSizes.length
=> 2 * (4 + 3) * 5 = 70
```

在上文中我们定义了很多的Map类型数据结构，为了以后维护方便，最好把他们放到一个单独的文件中。

新建一个文件_variables.scss。把之前定义的所有变量都放到这里维护。

在style.scss中通过**import**把它们引入进来。
```scss
@import './variables';
```

上面新建的文件名前面有个_， \_表示此文件将不会被编译为css文件，并且引用该文件的时候带不带_都可以。

### 总结
通过上面的介绍，你会发现，其实核心就是@each的用法，如果在配合一些@if @else条件语句，就可以很方便的构建一个非常轻量级的样式库。

如果你是利用构建工具(Gulp, Webpack等)编译，配合[autoprefixer](https://www.npmjs.com/package/autoprefixer)可以很方便的构建一个兼容各个浏览器的样式库。
