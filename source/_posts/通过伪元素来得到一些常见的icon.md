---
title: 通过伪元素来得到一些常见的icon
date: 2017-03-18 17:29:06
tags: CSS
---
通过伪元素`:before`和`:after`来得到一些常用的icon
<!-- more -->

```css
// caret
.icon_caret {
    display: block;
    margin: 5px 0 0 5px;
    width: 0;
    height: 0;
    line-height: 0;
    font-size: 0;
    border-width: 5px;
    border-style: solid;
    border-color: #9d9d9d transparent transparent transparent;
}

// >
.icon_gter{
    width: .16rem;
    height: .16rem;
    margin: 0 12px;
    display: block;
    position: relative;
    margin-top: 0;
    &:after{
        content: "";
        width: .08rem;
        height: .08rem;
        border-width: 1px;
        border-style: solid;
        border-color: transparent transparent rgba(0,0,0,0.3) rgba(0,0,0,0.3);
        -webkit-transform: rotate(-135deg);
        transform: rotate(-135deg);
        position: absolute;
        left: .05rem;
        top: .04rem;
	}
}
// <
.icon_lser{
    width: .16rem;
    height: .16rem;
    margin: 0 12px;
    display: block;
    position: relative;
    margin-top: 0;
    &:after{
        content: "";
        width: .08rem;
        height: .08rem;
        border-width: 1px;
        border-style: solid;
        border-color: transparent transparent rgba(0,0,0,0.3) rgba(0,0,0,0.3);
        -webkit-transform: rotate(-45deg);
        transform: rotate(-45deg);
        position: absolute;
        left: .05rem;
        top: .04rem;
        }
}

.triangle{
    width:0;
    height:0;
    border-width:20px;
    border-style:solid;
    border-color:transparent transparent red transparent;
}
```
注意每个border-color对应一个三角形