---
layout: post
title: Shell Regular Expression
categories: [Shell]
tags: [Regular Expression, Linux, Unix]
excerpt: "
在Unix系统的使用中经常会进行文本处理，不可变避免的会经常使用正则表达式，但在Unix里实际上有两种正则表达式——BRE(Basic Regular Expression)和ERE(Extended Regular Expression)。
"
#MathJax: true
---


* Contents
{:toc #toc_of_keans_blog}

# Shell里的正则表达式

在Unix系统的使用中经常会进行文本处理，不可变避免的会经常使用正则表达式，但在Unix里实际上有两种正则表达式——BRE(Basic Regular Expression)和ERE(Extended Regular Expression)，这将是本文主要介绍的内容。

## 正则表达式




字符|BRE/ERE| 模式含义
---|-------|---------
`\`  | Both  | 相当于转意符，用于取消后面字符的特殊意义。有时则表示后面字符具有特殊意义，如`\(...\)`与`\{...\}`
`.`  | Both  | 匹配任何单个字符，但NUL除外。独立程序也可以不匹配换行字符
`*`  | Both  | 匹配之前出现的字符任意个（或没有），以BRE来说，`*`出现在第一个字符不具有任何意义
`^`  | Both  | 在行首或字符串首，匹配紧接着的正则表达式。BRE：仅在正则表达式开头具有特殊含义；ERE：置于任何位置均有特殊含义
`$`  | Both | 与`^`类似，但是作用在行或字符串尾部
`[...]` | Both | 方括号表达式，匹配方括号内任一字符
`[^...]` | Both | 不匹配方括号内任一字符,
`[a-z0-9A-Z]` | Both | 在方括号使用区间
`\{n,m\}` | BRE | 区间表达式，匹配前面的字符n到m次
`\{n\}` | BRE | 匹配前面的字符n次
`\(...\)` |BRE| `...`所表示的模式存储在特殊的保留空间，一个正则表达式中，最可以有9个这样的模式，可以通过`\1`至`\9`来重复使用这些模式
`\n` | BRE | n 为1到9，需配合`\(...\)`使用
`{n,m}` | ERE | 与BRE的 `\{n,m\}`功能相同，也有`{n}`的用法
`+` | ERE | 匹配前面的表达式一个或多个，类似于`{1,+Inf}`
`?` | ERE | `{0,1}`
`|` | ERE | 相当于逻辑或，可匹配位于 `|`之前或者之后的正则表达式，优先级最低
`()` | ERE | 分组，括号里的正则表达式当成一个整体，如`(why)+`将匹配一个或多个连续的why

## POSIX 方括号表达式

- **字符集** 用`[:` 和`:]`将关键字括起来的为POSIX字符集
- **排序符号** 指将多个字符视为一个单位，使用`[.` 和`.]`括起来，其在系统使用的locale上各有其定义
- **等价字符集** 指出视为等值的一组字符，用`[=`和`=]`括起

**这三种构造都必须必须在方括号括表达式中使用，如：`[[:alpha:]]`匹配任一个英文字母字符**

**POSIX字符集**

类别 | 匹配字符 | 类别 | 匹配字符
----|---------|-----|--------
`[:alnum:]`| 字母与数字字符| `[:lower:]` | 小写字母
`[:alpha:]`| 字母字符 | `[:print:]` | 可打印字符
`[:blank:]`| 空格（space）和定位（tab）字符| `[:punct:]` | 标点字符
`[:cntrl:]`| 控制字符| `[:space:]` | 空白字符（whitespace）
`[:digit:]`|十进制数字字符 | `[:upper:]` |大写字母
`[:graph:]`| 非空格（nonspace）字符| `[:xdigit:]` | 十六进制数字

## BRE

- **在方括号表达式中所有其他的meta字符失去其特殊含义，故`[*\.]`将匹配于字面上的星号、反斜杠以及点号。对于`]-`加入集合，`]`只能放在最前面，`-`只能放在最前面或者最后面，如：`[]*.]`或`[]*.-]`**
- 另外POSIX指出，NUL字符（数值零，即C语言的`'\0'`也即`NULL`）不需要匹配。另有个别工具程序不许使用`.`点号meta字符或者方括号表达式来进行换行字符的匹配
- **后向引用：**即`\(...\)`和  `\n`结合起来使用的功能
- **BRE运算符优先级，由高到低**

运算符 | 表示意义
------|-------
`[..] [==] [::]` | 方括号表达式里的，字符排序、字符集和等值符号
`\meta_char` | 转意的meta字符
`[]` | 方括号表达式
`\( \) \n` | 子表达式与后向引用
`* \{ \}` | 前置单个字符重现的正则表达式
无符号（no symbol）| 连续
`^ $` | 锚点

## ERE
- 匹配单个字符时，ERE本质上与BRE一致。一个较有名的例外是在awk中,其`\`符号在方括号表达式内表示转意，因此要想匹配左方括号、连字符、右方括号或是反斜杠，应该使用`[\[\-\]\\]`
- ERE里没有后向引用
- `|` 交替，或，其优先级最低，如： `ab|cd|ef`将匹配`ab`或`cd`或`ef`
- **分组** `()`提供分组功能，如`((read|write)[[:space:]]*)+`表示匹配多个连续的read和write其中可能有空格分隔
- 在ERE中`^`和`$`永远有效，如`ab^cd`，`ef$gh`虽然不会匹配到任何东西，但是表达式有效，使用本意需转意。
- **ERE运算符优先级，由高到低**

运算符 | 表示意义
------|-------
`[..] [==] [::]` | 方括号表达式里的，字符排序、字符集和等值符号
`\meta_char` | 转意的meta字符
`[]` | 方括号表达式
`\( \) \n` | 子表达式与后向引用
`* +  ? {} ` | 前置单个字符重现的正则表达式
无符号（no symbol）| 连续
`^ $` | 锚点
`|` | 交替，或

## 正则表达式的扩展（注意不是ERE）
- `\< \>` 分别匹配单词开头与结尾
- **额外的GNU正则表达式运算符**


运算符 | 表示意义
------|-------
`\w` | 所有单词组成字符，等同于 `[[:alnum:]]`
`\W` | 所有非单词组成字符，等同于 `［^[:alnum:]_]`
`\< \>` | 匹配单词开头与结尾
`\b` | 匹配单词开头或结尾的空字符串
`\B` | 匹配两个单词之间的空字符串
`\'\`\` | 匹配emacs缓冲区的开始与结尾。GNU程序通常将它们视为`^ $`的同义

## 程序与正则表达式
**Unix 程序及其对应的正则表达式类型**

类型  | grep | sed | ed | ex/vi | more | egrep |       awk | lex
-----|------|------|----|------|------|-------|----|-----
**BRE** | YES(默认） | YES | YES| YES| YES | | |
**ERE** |grep -E|     |    |    |     | YES | YES | YES
`\< \>` | YES | YES | YES| YES| YES | | |
