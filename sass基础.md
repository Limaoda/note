## scss

#### 注释

```scss
/* 
 * 我是多行注释，会出现在编译后的css文件中
 * 描述
 */

// 我是单行注释，不会出现在编译后的css文件中

/*
 * 我是多行注释 
 * 插值 1 + 1 = #{1+1}
 */

/*! 这行注释即使在压缩模式下也会编译到 css 中 */

```

#### 文档注释(sassDoc)

```scss
/// @params {number} $num
/// @return {number} `$base`
```



#### 变量（variable）

```scss
/*
 * 使用$符号进行变量声明，告诉编译器这是一个sass变量，冒号表示赋值，相当于js的=符号，右边为一个css存在的属性值
 */
$red: #f00; //$color表示red这个css属性色值

div{
  color: $red;
}
```
等同于
```css
div{
  color: #f00;
}
```

#### css变量与sass变量的不同

```css
/* css的变量声明之后如果被重新声明，所有使用到该css变量的属性全部变更，而sass变量是不可逆的，也就是说声明变量之后使用到该变量的属性全部不变，变量被重新声明后，只会影响到后面的变量 */
```

#### sass变量不区分连字符和下划线

```scss
$font-color: #f20;
body{
  background-color: $font-color;
}
```

等同于

```scss
$font-color: #f20;
body{
  background-color: $font_color;
}
```

#### 配置模块变量

用 `!default` 定义的变量，可以在使用 `@use` 规则加载模块时配置。

module.scss

```scss
$font-color: #f20 !default;
body{
  background-color: $font-color;
}
```

在引用模块时，选择要自定义值的变量，忽略的变量则使用默认值：

```scss
// index.scss
@use 'module' with (
  $font-color: #f20;
);
```

#### 变量作用域

Index.scss

```scss
$font-color: #f20;
body{
  $font-color: #f8f;
  color: $font-color; // #f8f
}
p{
  color: $font-color; // #f20
}
```

##### 局部变量覆盖全局变量

```scss
$font-color: #f20;
body{
  $font-color: #f8f !global;
  color: $font-color; // #f8f
}
p{
  color: $font-color; // #f8f
}
```

#### 插值

用法：在 #{} 中放置一个表达式，通常用于将变量值插入字符串中

==插值永远返回一个不加引号的字符串==

```scss
$num20: 20;
.ml-#{$num20+20}{
  margin-left: #{$num20+20}px;
}
```

等同于

```css
.ml-40 {
  margin-left: 40px;
}
```

插值适用范围非常广泛，如下的场景均可以使用插值

- 选择器
- 属性名
- 自定义属性值
- CSS 的 `@` 语句中
- `@extends`
- CSS `@imports`
- 字符串
- 特殊函数
- CSS 函数名
- 保留注释（Loud comments ） `/* ... */`

```{}scss
/* 1+1 = #{1+1} */

// 自定义属性值
$direction: le#{'ft'};
// 选择器
.ml-#{(10+5)*20}{
  // 属性名
  margin-#{$direction}: #{(10+5)*20}px
}
```

#### @语句

##### @mixin和@include

mixin用于定义需要复用的样式块，include用于调用这些样式块

基本语法：

```scss
@mixin <name>{

/* 这里写上需要复用的样式 */

}

/* 带有形参的mixin */
/* 一个 mixin 定义时有多少个参数，那么在调用时必须传递相同数量的参数 */
@mixin <name>(arg1,arg2,...){
  /* 这里写上需要复用的样式 */
}

@include <name>;

/* 调用mixin并传入实参 */
@include <name>(arg1,arg2,...);
```

例子：

```scss
@mixin btn-style{
  background-color: #00f;
  color: #fff;
  &:hover{
    color: #00f;
    background-color: #fff;
  }
}

.submit-btn{
  padding: 10px;
  @include btn-style;
}

/************分割线**************/

@mixin btn-style($color,$bg-color){
  background-color: $bg-color;
  color: $color;
  &:hover{
    color: $bg-color;
    background-color: $color;
  }
}

.submit-btn{
  padding: 10px;
  @include btn-style(#fff,#00f);
}
```

