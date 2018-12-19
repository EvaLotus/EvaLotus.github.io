---
title: 邮件中的HTML
date: 2017-07-18T14:21:41.000Z
tags: HTML
---

马上要举办iOT大会，突然来了个急需求，要把邀请函做成HTML嵌在邮件里发送给参会嘉宾，心里想这有何难，三下五除二写完发给了小伙伴，真正放在邮件里看都傻眼了。

outlook客户端里：css统统不生效。

手机端显示更是惨不忍睹，各种手机表现形式完全不一样。

真是狠狠打自己的脸。

速google了一下，果然这种邮件内嵌html都是有固定的格式要求的，这种HTML Emial能否正常显示完全就取决于邮件客户端，而web技术发展了这么多年，这种邮件中的HTML却丝毫没有发展，各种邮件客户端都特别傲娇，简直可以称得上是面目全非。

一个准则就是：怎么原始怎么来。

1.不要再考虑css提取出来的问题，直接全部写在element style里。

不能使用reset实在太臃肿了，要保持一致性要写很多重复的代码，怀念现代的生活。

2.不要想flex布局，float布局，绝对定位啦！直接上年轻人都没听过的table布局。

```
<table align="center" border="1" cellpadding="0" cellspacing="0" width="600" style="border-collapse: collapse;">

　<tr>
　　<td> Row 1 </td>
　</tr>

　<tr>
　　<td> Row 2 </td>
　</tr>

　<tr>
　　<td> Row 3 </td>
　</tr>

</table>
```

先想好布局分为几部分，写在各个tr里。

3.图片是唯一可以引用的外部资源。其他的外部资源，比如样式表文件、字体文件、视频文件等，一概不能引用。

遵循上面3条规则，代码越写φ\(≧ω≦\*\)♪越臃肿了，但是最可怕的是发现，自己根本不懂table！
![一个很复杂的表格](/source/images/complex_table.png)

