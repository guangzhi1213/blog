---
title: "Powershell"
date: 2021-04-21T18:23:38+08:00
lastmod: 2021-04-21T18:23:38+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---


# powershell
## 管道命令
`ls | sort -Descending Name | Format-Table Name,Mode`
>通过ls获取当前目录的所有文件信息，然后通过Sort-Descending对文件信息按照Name排序，然后格式化输出  

## 重定向
1. `>` 覆盖 `>>` 追加
2. $HOME > 1.txt , Get-Content .\1.txt


# 交互式
## 运算
1. +,-,\*,/,%,(), 支持这几种运算方法
2. 不区分小数整数
3. 自动识别计算机容量单位，KB，MB，GB，TB，PB

## 执行外部命令 netstat ipconfig
netstat 查看网络接口
ipconfig 查看网络连接状态
route print 查看路由信息
添加环境变量：
`$env:Path=$env:Path+”%ProgramFiles%\Windows NT\Accessories”`  
默认输入一个字符串会输出，如果此字符串是一个命令或启动程序，在前面加`&` 可以执行或启动此程序

## 别名
`Get-Alias -name ls` ： 查询别名所指的真实cmdlet命令  
`ls alias:/Get-Alias` : 查看可用的别名;  
`dir alias: | where {$_.Definition.Startswith("Remove")}`
 查看所有以Remove打头的cmdlet的命令的别名  
> dir alias:获取的是别名的数组，通过where对数组元素进行遍历，$\_代表当前元素，alias的Definition为String类型，因为powershell支持.net，.net中的string类有一个方法Startswith。  

`Set-Alias -Name Edit -Value notepad` 创建自己的别名  
`del alias:Edit` 删除自己的别名
`Import-Alias alias.ps1` 导出别名，自己试试没成功，  
`Export-Alias -Force alias.ps1` 强制导入
### 定义函数别名
`function test-conn { Test-Connection  -Count 2 -ComputerName $args}`  
`Set-Alias tc test-conn`  
`tc localhost`  

## 执行脚本
1. 第一次执行进本可能会失败，默认禁止执行脚本 `set-ExecutionPolicy RemoteSigned`
2. 执行优先级： 别名 > 函数 > 命令 > 脚本 > 文件

# 变量
## 定义变量
1. ps不需要特意声明变量  
$name="hello world"  
$a=1  
$b=2  
$sum=$1+$b  
\# 输出
$name  
$a  
$b  
$sum  
2. 给多个变量赋值  
$a=$b=$c=123  
3. 交换变量的值  
`$value1,$value2=$value2,$value1`  
> 这个稍微比原先的寻找一个中间变量，来回交换要方便  

4. 查看正在使用的变量  
ls variable:
5. 查找变量  
ls variable:value*  
查找以value开头的变量
6. 验证变量是否存在  
Test-Path variable:a  
7. 删除变量  
ps退出后变量自动失效，一般不用；del variable:value1  
8. 变量专用命令  
Clear-Variable，*Get-Variable*，*New-Variable*，Remove-Variable，Set-Variable  
9. 变量写保护  
`New-Variable num -Value 100 -Force -Option readonly`  
只有在删除变量后，才可以重新恢复权限  
10. 权限更高的设置
`new-variable num -Value "strong" -Option constant` 一旦常亮声明，不可修改  
11. 变量描述  
new-variable name -Value "me" -Description "This is my name"  
可以通过Format-List 查看，ls Variable:name | fl *  
12. 自动化变量   $$,$?,$^,$\_,$Args,$ConsoleFileName,$Error,$Event,$EventSubscriber,$ExecutionContext,$False,$ForEach,$Home,$Host,$Input,$LastExitCode,$Matches,$MyInvocation,$NestedPromptLevel,$Null,$PID,$Profile,$PsCmdlet,$PsCulture,$PSDebugContext,$PsHome,$PSScriptRoot,$PsUICulture,$PsVersionTable,$Pwd,$Sender,$ShellID,$SourceArgs,$SourceEventArgs,$This,$True, 诸多  

### 环境变量  
del env:windir  
$env：中的环境变量只是机器环境变量的一个副本,在此次修改了之后只会影响当前shell，并不会影响系统。  
```
$+圆括号+表达式 构成的变量属于子表达式变量，这样的变量会先计算表达式，然后把表达式的值返回  
PS> "The size of Powershell_Cmdlets.html is $($file.Length)"  
```
### 变量作用域  
`全局、当前、私有和脚本`  

指定类型定义变量 [byte]$b=101  ,$b=256会报错，最大255  

### xml文档  
LogTest.xml  
```
<logotest>
  <extensions>
    <e>.exe</e>
    <e>.dll</e>
  </extensions>
  <files>
    <f></f>
  </files>
  <dirs></dirs>
</logotest>
```  
[ XML ]$xml=(Get-Content .LogoTestConfig.xml)  
 $xml.LogoTest.Extensions.E  
.exe  
.dll  
Powershell 默认支持的.NET类型如下。  
[array],[bool],[byte],[char],[datetime],[decimal],[double],[guid],[hashtable],[int16],[int32],[int],[int64],[long],[nullable],[psobject],[regex],[sbyte].[scriptblock],[single],[float],[string],[switch],[timespan],[type],[uint16],[uint32],[uint64],[ XML ]  
## 强类型数组  
使用类型名和一对方括号作为数组变量的类型  
若赋值不为此类型，则报错  
```
[int[]] $nums=@()
$nums+=2012
$nums+=12.3

$num+="hello world" 则报错  

需要注意的ui添加小数不会报错，但是会自动转换为整数； 若为负数，则是（-1.1=-1，-1.5=-2，-0.9=-1）好尴尬的四舍五入  
```
## 变量的幕后管理  
Get-Variable  

## 数组
$name="lisa","jerry","bob"

* 数组的多态  

$array=1,"2012世界末日",([System.Guid]::NewGuid()),(get-date)  

* 空数组  
$abc=$()  
* 单元素数组  
$abc,"boss"  

## 数组访问  
1. 单个元素访问  
$book = "lubinxun","unix高级编程","鸟哥的私房菜"  
$book[0]  
$book[()$book.count-1)]  
$book[-1]:表示最后一个元素  
$book[i-1]  
2. 多个元素  
$result=ls  
$result[0,3,12]  
3. 逆向输出  
$result[()$result.count)..0]  
4. 数组添加或删除元素  
增加元素实质是把创建了新的数组，因为元素是顺序存储的，  
$book+="python高级编程"  
$book.count  
删除元素  
没有此方法??  
$num=$num[0..1]+=$num[3]  

## 哈希表  
1. 创建数组（xx数组）  
@() 数组  
@{} 哈希表  
$stu=@{ Name = "小明";Age="12";sex="男" }  
引用：  
$stu["Name"]，$stu["Age"],$stu["sex"]  
$stu.Values  
2. 创建带数组的哈希表  
$stu=@{ Name = "小明";Age="12";sex="男";Books="三国演义","围城","哈姆雷特" }  
3. 插入新的键值  
$stu=@{}  
$stu.Name="小明"  
$stu.Books="三国演义","围城","哈姆雷特"  
4. 哈希表更新和删除  
$stu.Name="赵强"  
$stu.Remove("Name")，删除  
5. 哈希表格式化输出  
```
Expression:绑定的表达式
Width:列宽度
Label:列标题
Alignment:列的对齐方式
```
$column1 = @{expression="Name"; width=30;label="filename"; alignment="left"}  
$column2 = @{expression="LastWriteTime"; width=40;label="last modification"; alignment="right"}  
ls | Format-Table $column1, $column2  
就可以格式化输出了  
## 复制数组  
$chs=@("A","B","C")  
$chsBak=$chs  
两份变量引用同一份数据；  
$chsNew=$chs.Clone()  
克隆一份；  

# 管道  
1. 等待错误：  
Get-Content d:\\1.txt -wait | where { $_  -match "WARNING" }  
> 使用管道
ls | Sort-Object -Descending Name | Select-Object Name,Length,LastWriteTime | ConvertTo-Html | Out-File ls.html  

2. 使用管道进一步执行的命令  
<pre>
Compare-Object: 比较两组对象。
ConvertTo-Html: 将 Microsoft .NET Framework 对象转换为可在 Web 浏览器中显示的 HTML。
Export-Clixml: 创建对象的基于 XML 的表示形式并将其存储在文件中。
Export-Csv: 将 Microsoft .NET Framework 对象转换为一系列以逗号分隔的、长度可变的 (CSV) 字符串，并将这些字符串保存到
一个 CSV 文件中。
ForEach-Object: 针对每一组输入对象执行操作。
Format-List: 将输出的格式设置为属性列表，其中每个属性均各占一行显示。
Format-Table: 将输出的格式设置为表。
Format-Wide: 将对象的格式设置为只能显示每个对象的一个属性的宽表。
Get-Unique: 从排序列表返回唯一项目。
Group-Object: 指定的属性包含相同值的组对象。
Import-Clixml: 导入 CLIXML 文件，并在 Windows PowerShell 中创建相应的对象。
Measure-Object: 计算对象的数字属性以及字符串对象（如文本文件）中的字符数、单词数和行数。
more: 对结果分屏显示。
Out-File: 将输出发送到文件。
Out-Null: 删除输出，不将其发送到控制台。
Out-Printer: 将输出发送到打印机。
Out-String: 将对象作为一列字符串发送到主机。
Select-Object: 选择一个对象或一组对象的指定属性。它还可以从对象的数组中选择唯一对象，也可以从对象数组的开头或末尾选
择指定个数的对象。
Sort-Object: 按属性值对象进行排序。
Tee-Object: 将命令输出保存在文件或变量中，并将其显示在控制台中。
Where-Object: 创建控制哪些对象沿着命令管道传递的筛选器。
</pre>
3. 转换成文本  
> 查看当前以i打头的进程，并显示进程的名字和其它以”pe”打头，以”64″结尾的进程  
`Get-Process i* | Format-Table Name,pe*64`

4. 排序和分组管道结果  
http://www.pstips.net/powershell-sort-and-group-pipeline-results.html  
### 过滤管道结果  
1. 筛选管道结果中的对象  
查看元素属性: `Get-service | Select-Object -First 1 | Format-List *`
`Get-service | Select-Object -First 1 | Get-Member -MemberType`  
查看正在运行的程序:`get-service | Where-Object {$_.Status -eq "Running"}`  

2. 限制对象数量  
get-process | sort -Descending cpu | select -First 5  
3. 通过管道处理所有结果  
ls | ForEach-Object {"文件名：{0} 文件大小{1}KB: " -f $_.Name,
($_.length/1kb).tostring()}  
4. 删除重复对象  
Get-Unique：  
ls | foreach{$\_.extension} | Sort-Object |Get-Unique  
5.
