## css 相关
### 1. 鼠标事件失效
```css
.lose-efficacy {
  pointer-events: none;
}
```
### 2. 文本两端对齐
```css
.justify {
  text-align: justify;
  text-justify: distribute-all-lines;  /* ie6-8 */
  text-align-last: justify;  /* 一个块或行的最后一行对齐方式 */
  -moz-text-align-last: justify;
  -webkit-text-align-last: justify;
}

```
### 3. 不换行、自动换行、强制换行
```css
/* 不换行 */
.nowrap {
  white-space: nowrap;
}
/* 自动换行 */
.auto-nowrap {
  word-wrap: break-word;
  word-break: normal;
}
/* 强制换行 */
.wrap {
  word-break: break-all;
}
```
### 4. 超出部分显示省略号
```css
/* 单行文本 */
.single-ellips {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 多行文本 */
.multi-ellips {
  width: 100%;
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;   /* 行数 */
  word-break: break-all;
}
```
### 5. 禁止用户选择
```css
.forbidden {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
```
### 6. 识别标签中的 '\n' 并实现换行
```css
div {
  white-space: pre-line;
}
```
### 7. 消除transition闪屏
```css
.wrap {
  -webkit-transform-style: preserve-3d;
  -webkit-backface-visibility: hidden;
  -webkit-perspective: 1000;
}
```
### 8. 使用caret-color改变光标颜色
```css
input{
  color: #fff;
  caret-color: green;
}
```

### 9. 使用transform硬件加速
```css
.elem {
  transform: translate3d(0, 0, 0); /* translateZ(0)亦可 */
}
```
### 10. 使用attr()抓取data-*
```html
<div data-msg="我是谁，我在哪儿" class="tips">提示信息</div>
```
```css
.tips {
  &::after {
    content: attr(data-msg);
  }
}
```
### 11. 使用pattern、:valid和:invalid校验表单
```html
<input type="text" placeholder="请输入名字(1到10个中文)" pattern="^[\u4e00-\u9fa5]{1,10}$" required>
```
```css
input {
  &:valid {
    border-color: green;
    box-shadow: inset 5px 0 0 green;
  }
  &:invalid {
    border-color: red;
    box-shadow: inset 5px 0 0 red;
  }
}
```
