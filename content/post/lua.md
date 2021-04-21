---
title: "Lua"
date: 2021-04-21T18:26:27+08:00
lastmod: 2021-04-21T18:26:27+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

## lua

- [Lua原来这么好用 | 罗道文的私房菜](http://luodw.cc/2017/03/24/lua/#more)
- [Lua简明教程 | | 酷 壳 - CoolShell](https://coolshell.cn/articles/10739.html)
- 

## lua by example

## lua

### 数据类型

1. locale 定义局部变量
1. string, function, boolean, number, nil
1. type("hello world") , 判断数据类型
1. string 连接用 ..
1. string [[]], [=[]=] 第n（n个等括号）级正长括号， 并且\n 不转义， 长括号可以包含长括号
1. math.floor(向下取整) math.ceil(向上取整)
1. table表, 关联数组 {1,2,3,4}, s={a=1, b=2, c=3, d={"jack","scoot","gary"}} 取用 s.a, s["a"], s[1],s.d[1]
1. function 函数
    1. local function foo(); local a = foo
    1. function foo();
    1. foo = function();
    1. local foo = function()

### 表达式

1. 算数表达式, + - * / ^ %
1. 关系运算符 <, >, <=, >=, ==, ~=
1. 逻辑运算符 and, or, not;  false 和nil 为假， 其余为真
1. 优先级
    1. ^
    1. not, #, -
    1. *, /, %
    1. +, -
    1. ..
    1. <, >, <=, >=, ==, ~=
    1. and
    1. or

### 控制结构

1. if
    1. if then else end
    1. if then elseif then end
1. for; 如果循环无上限，可以使用 math.huge
    1. for_numeric 数字
    1. for_generic 泛型
        1. 迭代文件中每行的（io.lines）
        1. 迭代 table 元素的（pairs）
        1. 迭代数组元素的（ipairs）
        1. 迭代字符串中单词的（string.gmatch）
    1. 哈希表的遍历本身也不会有数组遍历那么高效,ipairs 比 pairs 高效
1. while; while boolean do ; xxx ; end
1. repeat until
1. break, return; 没有continue

```lua
-- reverse table
local days = {
   "Monday", "Tuesday", "Wednesday", "Thursday",
   "Friday", "Saturday","Sunday"
}

local revDays = {}
for k, v in pairs(days) do
  revDays[v] = k
end
```

### 函数

1. 参数
    1. 传递值， 不会改变外部变量的值
    1. 形参跟实参不等时  从左到右， 或被丢弃，或被赋值nil
    1. 支持变长参数 形参写成 ...
    1. 引用传递， 除了 table 是按址传递类型外，其它的都是按值传递参数

### string库

1. string.upper(s)
1. string.lower(s)
1. string.len(s)
1. string.byte()
1. string.sub()
    1. string.sub("hello,world",3,7)
    1. string.sub("hello,world",3)
1. string.byte()
    1. print(string.byte("abc", 1, 3))
    1. print(string.byte("abc", 3)) -- 缺少第三个参数，第三个参数默认与第二个相同，此时为 3
    1. print(string.byte("abc"))    -- 缺少第二个和第三个参数，此时这两个参数都默认为 1
1. string.char(...)
    1. print(string.char(96, 97, 98))
    1. print(string.char()) -- 参数为空，默认是一个0，输出空
1. string.find(s, p [, init [, plain]]); 
    1. s 字符串, 匹配 p 字符串,  init 默认为 1，并且可以为负整数表示从 s 字符串的 string.len(s) + init 索引处开始, plain默认为 false，当其为 true 时，只会把 p 看成一个字符串对待
    1. 返回 p 字符串在 s 字符串中出现的开始位置和结束位置
    1. 匹配失败，则返回 nil
    1. 

```lua
-- string.find()
local find = string.find
print(find("abc cba", "ab"))
print(find("abc cba", "ab", 2))     -- 从索引为2的位置开始匹配字符串：ab
print(find("abc cba", "ba", -1))    -- 从索引为7的位置开始匹配字符串：ba
print(find("abc cba", "ba", -3))    -- 从索引为6的位置开始匹配字符串：ba
print(find("abc cba", "(%a+)", 1))  -- 从索引为1处匹配最长连续且只含字母的字符串
print(find("abc cba", "(%a+)", 1, true)) --从索引为1的位置开始匹配字符串：(%a+)

-->output
1   2
nil
nil
6   7
1   3   abc
nil
```

1. string.format(formatstring, ...), 按照格式化参数 formatstring，返回后面 ... 内容的格式化版本
1. string.match(s, p [, init])
    1. string.match 目前并不能被 JIT 编译，应 尽量 使用 ngx_lua 模块提供的 ngx.re.match 等接口
1. string.gmatch(s, p)
    1. 前并不能被 LuaJIT 所 JIT 编译，而只能被解释执行。应 尽量 使用 ngx_lua 模块提供的 ngx.re.gmatch 等接口
1. string.rep(s,n), 返回字符串s的n次拷贝
1. string.gsub(s, p, r [, n]), 将目标字符串 s 中所有的子串 p 替换成字符串 r
    1. 可选参数 n，表示限制替换次数。返回值有两个，第一个是被替换后的字符串，第二个是替换了多少次。
1. string.reverse (s), 翻转字符串
