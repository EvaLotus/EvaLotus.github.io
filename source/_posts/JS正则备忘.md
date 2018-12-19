---
title: JS正则备忘
date: 2017-12-14 12:15:03
tags: javascript
---
#
## JS正则

#### 基础知识

##### 定头定尾（^,$）

^abc表示以此开头，定头
abc$以此结尾，定尾
^abc$以abc开头和结尾，定头定尾，**完整匹配**
abc匹配：abc

##### ?,\*,+,{2,},{2} 字符的个数

ab?匹配：（a后面跟着0个或1个b）
`ab*`匹配：（a后跟着0个或多个b）
ab+匹配：（a后跟着至少一个b）
a?b+$匹配：末尾是0或1个a和至少一个b结尾
ab{2}匹配：abb
ab{2,}匹配：a跟着至少2个b
ab{3,5}匹配：a跟着3到5个b

##### \|或

（）分组

.表示任意字符

a\|b匹配：a或b
\(b\|cd\)ef匹配：bef或cdef
`(a|b)*c` a或b出现任意次 后跟一个c
`a.[0-9]`匹配：a后任意字符和一个数字
^.{3}$匹配任意三个字符

##### \[\]表示包含\[\]的

\[ab\]匹配：a或b
\[a-z\]匹配：a到z中一个
^\[a-zA-Z\]匹配：开头是字母
`[0-9]%`匹配：%前一位数字
`[a-zA-Z0-9]$` 匹配：末尾是数字或字母

#### ^出现在\[ \]括号中表示匹配不出现某字符

`[^XXX]`表示不希望出现XXX

`%[^a-zA-Z]%`表示两个%间不希望出现字母

#### 元字符和转义

\(\[{\^$\|\)?\*+.}这些都是元字符，有特殊的功能，如果需要匹配字符串中的这些字符，必须要加转移符\

但如果在\[ \]中则不需要加转义字符

#### 一些简单表达

`\W`匹配：非字母数字或下划线，等价于[^a-zA-Z0-9]
`\w`匹配：字母数字，等价于\[a-zA-Z0-9\]
`[._]`匹配：特殊字符集
`\D`匹配：非数字,等价于[^0-9]
`\d`匹配：数字，等价于\[0-9\]
`\b`匹配：一个单词的边界
`\B`匹配：非一个单词的边界
`\s`匹配： 任何空白符,等价于`[ \t \n \r \f \v]`
`\S`匹配： 任何非空白符,等价于`[^\ t \ n \ r \ f \ v]`
.匹配：除了换行符之外的任意字符，等价于`[^\n]`：

^既表示定头，在`[]`中又表示不允许出现

`/^[a-zA-Z]{1}([a-zA-Z0-9]|[._]){4,19}$/`
第一位是字母，后面是4到19个数字或字母或任意字符串或\_

`^\d+$`匹配：正整数+0还有0999这样的
`^[0-9][1-9][0-9]`匹配：正整数还是有00100这样的
^\(\(-\d+\)\|\(0+\)\)负整数还有000这样的
^\[\W-\]+\(.\[\w-\]+\)\*@\[\W-\]+\(.\[\w-\]+\)+$

https?//\[\w./\]+可以出现数字字母和-及，或/

### js中的正则

new RegExp\('pattern',\[flags\]\)

#### RegExp的实例属性：

global:对应g flage，全局匹配

ignoreCase:对应i flag，忽略大小写匹配

multiline:对应 m flag，跨行多行匹配

source:pattern的字面值，/\d+/g.source=="\d+"// false

lastIndex:开始搜索下一个匹配项的字符位置，从0开始。主要在exec中会有变化

可以直接写字符串，也可new RegExp，或者不带引号匹配：在/中/
{}等价于&gt;=和&lt;=

js中的正则：

```
var pattern=new RegExp(/\d+/);
// flags:g,i,m,全局，忽略大小写，多行查找
var pattern=/\d+/gi;
```

#### RegExp的实例方法：

#### exec

```
var matches = /.ay(.hs)(.ds)/g.exec('syeuwyaydhsidsh2');

// ["yaydhsids", "dhs", "ids", index: 5, input: "syeuwyaydhsidsh2", groups: undefined]

// 返回的是有index属性和input属性的数组

// index是匹配的字符串的首字符的index

// input是输入的字符串

// matches[0]是匹配的到字符串

// matches[1,2...]是捕获组即(XXX)捕获到的字符串$1,$2
// 即使有g全局匹配，每次也只返回一个匹配项
var text='cat,fat,hat,bat'
var pattern=/.at/g;
var matches = pattern.exec(text); // cat
matches = pattern.exec(text); // fat
// pattern必须要带有g标志，且pattern和text都必须要保存在对象中，才可以多次执行，匹配下一个
```

#### match

```
var matches=str.match(regexp)
// 得到的结果和exec完全一致
var str = 'For more information, see Chapter 3.4.5.1';
var re = /see (chapter \d+(\.\d)*)/i;
var found = str.match(re);
// ["see Chapter 3.4.5.1", "Chapter 3.4.5.1", ".1", index: 22, input: "For more information, see Chapter 3.4.5.1"];
```

#### replace

```
str.replace(regexp|substr, newSubstr|function)
// 注意不修改原来的字符串，而是返回处理后的字符串

function replacer(match, p1, p2, p3, offset, string) {
// p1 is nondigits, p2 digits, and p3 non-alphanumerics
return [p1, p2, p3].join(' - ');
}
var newString = 'abc12345#$*%'.replace(/([^\d]*)(\d*)([^\w]*)/, replacer);
console.log(newString); // abc - 12345 - #$*%

// 交换单词，主要用到了(XX)捕获组,$1
var re = /(\w+)\s(\w+)/;
var str = 'John Smith';
var newstr = str.replace(re, '$2, $1');
console.log(newstr); // Smith, John
```

#### test

```
/.at/g.test('cat')// true 返回是否匹配
```

### 高级用法

#### lookahead

js只支持前瞻（也称零宽断言）lookahead向前查找，不支持后顾lookbehind

**注意正则是从尾部向头部解析的**

判断一个单词字符之后是否是数字（正向前瞻），是的话，则符合匹配进行替换

```
// exp1(?=exp2) 正向前瞻，匹配后面满足exp2的exp1
// 匹配后面满足\d的前面满足\w的\w对应的字符
'a2*3'.replace(/\w(?=\d)/g, "X"); // 'X2*3'

// exp1(?=exp2) 负向前瞻，匹配后面不满足exp2的exp1
// 匹配后面不满足\d的前面满足\w的\w对应的字符
'a2c*3'.replace(/\w(?!=\d)/g, "X"); // XXX*X"

// 要注意前瞻是非捕获性分组，不能通过$1来获取
'a2c*3'.replace(/\w(?!=\d)/g, "$1"); // $1$1$1*$1
```

后顾不支持就不写了

注意正则是从尾部向头部解析的，前瞻是指正则从后往前解析，后顾则是从前往后解析。

从后往前解析的意思是先满足\(\)中的exp2，再看前面的exp1是否符合

#### 贪婪模式

/\d{3,6}/一般匹配6，/\d{3,6}?/在量词后加上?即可非贪婪模式

引用：$捕获到分组的内容
$1对应每个\(\)分组的内容

#### ES6中的扩展

### 常用正则：

在sublime中输入commd+ shift+enter就可以替换为\r\n

\[^\x00-\xff\] 匹配中文

```
var str = "https://www.runoob.com:80/html/html-tutorial.html";
var patt1 = /(\w+):\/\/([^/:]+)(:\d*)?([^# ]*)/;
matches = str.match(patt1);
matches[0] // 得到匹配的url
matches[1] // scheme
matches[2] // domain
matches[3] // port
matches[4] // path
```


