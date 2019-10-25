# 移动端 H5 开发

## 前言

在移动端开发的过程中，webview 在安卓、IOS 中往往会发生一些不可思议的 bug，这篇主要摘录开发中遇到的 Bug 以及其解决方法。

### 1. viewport 模板

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta
      content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"
      name="viewport"
    />
    <meta content="yes" name="apple-mobile-web-app-capable" />
    <meta content="black" name="apple-mobile-web-app-status-bar-style" />
    <meta content="telephone=no" name="format-detection" />
    <meta content="email=no" name="format-detection" />
    <title>标题</title>
    <link rel="stylesheet" href="index.css" />
  </head>
  <body>
    这里开始内容
  </body>
</html>
```

### 2. 浏览器后退不刷新

webview 中比较常见，当点击后退时页面以缓存形式出现

```js
window.onpageshow = function(evt) {
  if (evt.persisted) {
    document.body.style.display = "none";
    location.reload();
  }
};
//  onpageshow每次页面加载都会触发，无论是从缓存中加载还是正常加载，和onload的区别点
// persisted页面是否从缓存中读出
```

## 3. input 相关

### 3.1. placeholder 文本位置偏上的情况

```css
input {
  line-height: normal;
}
```

### 3.2. 设置 placeholder 字体的大小

```css
::-webkit-input-placeholder {
  font-size: 12px;
}
```

### 3.3. 清除输入框内阴影

```css
input {
  -webkit-appearance: none;
}
```

### 3.4 去除 type 为 number 箭头样式

```css
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none !important;
  margin: 0;
}
```

### 4. 文本缩放（字体加粗不一致）问题

```css
body {
  -webkit-text-size-adjust: 100%;
  -ms-text-size-adjust: 100%;
  text-size-adjust: 100%;
}
```

### 5. 禁止用户选择

```css
body {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
```

### 6. 禁止保存或拷贝图像

```css
img {
  -webkit-touch-callout: none;
}
```

### 7. 优化渲染性能，开启硬件加速

```css
elem {
  -webkit-transform: translate3d(0, 0, 0);
  -moz-transform: translate3d(0, 0, 0);
  -ms-transform: translate3d(0, 0, 0);
  transform: translate3d(0, 0, 0);
}
```

## IOS 相关

### 1. IOS 键盘字母输入，默认首字母大写的解决方案

```html
<input autocapitalize="off" autocorrect="off" />
<!-- autocapitalize: 自动大小写
    autocorrect: 纠错
    autocomplete: 自动记录输入值
-->
```

### 2. IOS 不会触发行内元素点击事件

```css
cursor: pointer;
```

### 3. IOS 中 location.href 跳转页面空白

```js
setTimeout(() => {
  window.location.href = "www.xxx.com";
}, 0);
```

### 4. 手机号识别（IOS）

1. 关闭电话号码的自动识别

```html
<meta name="format-detection" content="telephone=no" />
```

2. 开启电话功能：

```html
<a href="tel:13516161012">13516161012</a>
```

3. 开启短信功能：

```html
<a href="sms:10086">10086</a>
```

## 相关文档

### 1. [移动端 web 开发技巧](http://liujinkai.com/2015/06/06/mobile-web-skill/)
