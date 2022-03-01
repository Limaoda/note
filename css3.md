#### 变量

```css
/* css变量声明符号为--,声明时需要制定一个规则作用域，通常为伪根元素:root，可作用至全局 */
:root{
  --main-bg-color: #fef8ef;
}

body{
  /* 使用css变量 */
  background-color: var(--main-bg-color);
}
```

有如下的html结构

```html
<div class="one">
  <div class="two">
    <div class="three"></div>
    <div class="four"></div>
  </div>
</div>
```
#### css3的变量具有继承性
```css
.one{
  --main-color: #ccc;
}
.two{
  background: var(--main-color);
}
/* class为three的元素其父元素为class为two的元素，当使用到变量时，自己的作用域找不到会去父级作用域找，最终找到父级作用域中存在值为#ccc的--main-color变量 */
.three{
  background: var(--main-color);
}
```

#### var函数的备用值

```css
.two{
  background: var(--main-color,pink);
}
/* class为two的元素使用到了--main-color变量，该变量在自己以及父级作用域找不到相应的声明，var函数的第二个参数为备用值，当第一个参数使用的变量找不到时会使用第二个参数（备用值）作为变量的值 */
.three{
  background: var(--main-color);
}
```

相当于

```css
.two{
  --main-color:pink;
  background: var(--main-color);
}
/* class为two的元素使用到了--main-color变量，该变量在自己以及父级作用域找不到相应的声明，var函数的第二个参数为备用值，当第一个参数使用的变量找不到时会使用第二个参数（备用值）作为变量的值 */
.three{
  background: var(--main-color);
}
```

#### 无效的var变量

```css
:root{
  --font-size: 16px;
}

.one{
  color: var(--font-size);
}

/* 浏览器首先会将变量--font-size正常解析为如下 */
.one{
  color: 16px;
}

/* 对于color属性来说16px是一个非法属性值，因此浏览器接下来会去判断color属性是否有继承性，如果有，会去找父级元素的color值，如果最顶层的元素都没有进行color的声明，则使用默认的color属性值black */
.one{
  color: black; /* 默认值 */
}

```

#### javascript操纵css变量

html

```html
<div class="my-class">
  我是文本
</div>
```

css

```css
.my-class{
  --font-size: 18px;
  font-size: var(--font-size);
}
```



```javascript
const $myClass = document.getElementsByClassName("my-class")[0]
// 获取一个 Dom 节点上的 CSS 变量
$myClass.style.getPropertyValue("--font-size") // '18px'

// 获取任意 Dom 节点上的 CSS 变量
getComputedStyle($myClass).getPropertyValue("--font-size");

// 修改一个 Dom 节点上的 CSS 变量
$myClass.style.setProperty("--font-size", '30px');

```
