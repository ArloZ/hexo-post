---
title: MarkDown 标记语法简介
date: 2014-05-11 17:21:53
tags: Markdown markup git
category: 杂七杂八
---

[markdown](http://zh.wikipedia.org/zh-cn/Markdown)作为一种轻量级的标记语言，十分的易读易写。正苦于没有足够简单的文档编辑工具，经朋友的简介也开始用之。

下面就对其标记语法做一下简单的记录。


## 多级标题

## title2    `##title2`

### title3   `###title3`

#### title4   `###title4`

同时也可使用三个或三个以上的`=`表示一级标题;
使用三个或三个以上的`-`表示二级标题
 
## 引用

`>`用于生成一段引用

>这是一段引用

## 列表

可使用`+ - *`符号之一加空格生成无序列表

* list1     `* list1`
- list2     `- list2`
+ list3     `+ list3`

使用`1. 2. 3. ......`之一加空格生成有序列表

1. list1    `1. list1`
2. list2    `2. list2`
3. list3    `3. list3`

## 代码区块


使用`'`符号(ESC下方的按键)包裹一段文字可生成代码块

通过一个tab缩进生成一个代码块

``` cpp
// 通过tab缩进生成代码块
int main(){
    int i=0;
    printf("hello world\n");
}
```

## 分割线


三个或三个以上的 `---` or `***` 可生成一条分割线

## 链接


一个链接由一对方括号和一对圆括号构成：`[link](http://qlzhang.github.io "Title")`

定义一个链接在多处使用

```
[id]: http://arloz.github.io "Title"
[link1][id]
[link2][id]
```
其中id定义的冒号后需有一个空格

## 强调


使用`_`或`*`包裹一段文字可以生成`<em>`效果的 _强调_

可使用2个或3个`_`或`*`表示 __加重强调__ 和 ___特别强调___

## 图片

```
![Alt text](/path/to/img)
```
在一个链接前加上感叹好可在文档中插入图片



