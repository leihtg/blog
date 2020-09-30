---
layout: post
title: html页面中鼠标选中复制
categories: JavaScript
tags: JavaScript DomEvent
author: leihtg
---

* content
{:toc}

经常复制网页内容时，鼠标选中一大片，然后按ctrl+v，按的手疼，挺麻烦的。最气人的是有时候还复制不上得多按几次，这就让人受不了了。
于是想到了用dom的事件监听功能，让js自动去复制，这就方便多了。



```javascript
// 释放鼠标处理函数
function mouseUp() {
    var text = "";
    if (window.getSelection) {
        text = window.getSelection().toString();
    } else if (document.selection && document.selection.type != "Control") {
        text = document.selection.createRange().text;
    }
    if ("" != text) {
        console.log(text);
		document.execCommand('Copy');
    }
}

document.addEventListener("mouseup", mouseUp, true);
```