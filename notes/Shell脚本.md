# 编写shell脚本

## 编写第一个Shell脚本

### **什么是Shell脚本？**

​	shell脚本就是包含一系列命令的文件。shell读取这个文件，然后执行这个命令，就好像命令是直接输入命令行中一样。

### **如何编写shell脚本？**

1. 编写脚本
2. 是脚本可执行
3. 将脚本放置在shell能够发现的位置

**脚本的格式**

```shell
#!/bin/bash
#this is out first script
echo 'hello world'
```

1. 脚本的第一行有特殊意义："#!" 称为释伴（shebang）用于告诉操作系统，执行后面的脚本应该使用的解释器名字。
2. \#后面的所有内容都会被忽略。
3. echo 'hello world' 是一个命令，用于输出hello world

**可执行权限**

我们使用chmod命令，来让脚本可执行。

对于脚本，有两种常见的权限设置：

​	755---每个人都可执行

​	700---只有脚本的所有人才能执行

注意：为了能够执行脚本，它必须是可读的。

**脚本的位置**

执行名为shell001的脚本：

```shell
./shell001
```

我们需要显示的指定脚本的路径。

如果没有显示的指定脚本的路径，则系统在查找一个可执行程序时候，需要搜索一系列的目录。目录列表存放在PATH的环境变量中。如果我们把脚本放在PATH中的任何一个目录中，则不需要指定路径也可以执行。

注意：我们也可以把脚本的当前路径添加进PATH环境变量中。（添加之后，记得重新加载配置文件）

## 启动一个项目

### 最小的文档

```shell
#!/bin/bash
#这是一个简单的文档
echo "<HTML>"
echo "	<HEAD>"
echo "		<TITLE>Page Title</TITLe>"
echo "	</HEAD>"
echo "	<BODY>"
echo "		Page body"
echo "	</BODY>"
echo "</HTML>"
```

通过更改脚本权限以及运行得到一个文档结构的输出。

```
<HTML>
        <HEAD>
                <TITLE>Page Title</TITLe>
        </HEAD>
        <BODY>
                Page body
        </BODY>
</HTML>
```

然后我们将多个echo命令整合成一个:

```shell
#!/bin/bash
#这是一个简单的文档 

title="这是我的新标题"
body="这是我自定义的主体内容"

echo "<HTML>
        <HEAD>
                <TITLE>$title</TITLE>
        </HEAD>
        <BODY>
                <H1>$body<H1>
        </BODY>
</HTML>"
```

输出：

```
<HTML>
        <HEAD>
                <TITLE>这是我的新标题</TITLE>
        </HEAD>
        <BODY>
                <H1>这是我自定义的主体内容<H1>
        </BODY>
</HTML>
```

1. 这里一个带引号的字符串可以包含换行符，因此也就可以包含多个文本行。
2. 我们添加了两个变量（title、body），这样就可以利用参数扩展，将该字符串放置到多个地方。

### 创建变量和常量

#### **如何创建变量？**

```shell
$ foo="yes"
$ echo $foo
#得到输出为：yes
$ echo $fool
#得到输出为：（空）
```

1. shell遇到一个变量的时候，会自动创建这个变量。这与大多数程序中，使用一个变量前先声明或者定义有所不同。
2. 变量的命名规则：
   - **由**字母、数字、下划线**组成**。
   - **第一个字符**必须是字母或下划线。
   - 名称中**不允许**空格和标点。

#### **常量**

常量其实就是一个有名称和确定值的变量。

区别在于常量的值不会发生变化。

<u>较为普遍的约定是，我们使用大写字母表示常量，使用小写字母表示变量。</u>

**强制常量不发生变化的方法**

```shell
declare -r TITLE="Page Title"
#使用带有 -r（只读）的declare内置命令来实现。
#shell将阻止一切后续向TITLE的赋值。很少用。
```

#### **为常量和变量赋值**

1. 变量的赋值方式：variable=value（变量名、等号、值之间<u>不能有空格</u>）
2. shell并不关心变量的值的数值类型，它都会当作字符串。
3. 可以通过 -i选项的declare命令，强制shell将变量限制为整型数值。但是，这样如同将变量设置为只读一样，很少使用。

```shell
a=z
#将字符串“z”赋值给变量a
b="a string"
#嵌入的空格，必须被引号扩起来
c="a string and $b"
#可以带有其他扩展（比如赋值语句）
d=$(ls-l foo.txt)
#可以是命令的结果
e=$((4*8))
#可以是算数扩展
f="\t\ta string\n"
#可以带有转义序列（制表符、换行符）
g=5 h="hello"
#可以在一行给多个变量赋值
```

在扩展期间，变量名称可以使用"{}"扩起来。这在变量名因为周围的上下文变得不明确时候，很有用。

```shell
$ filename="myfile"
#创建变量filename，值为myfile
$ touch $filename
#通过变量创建文件
$ mv $filename $filename1
#将文件名称从myfile改为myfile1
#失败，因为shell会将filename1也看作一个变量，但是并没有这个变量。
$ mv $filename ${filename}1
#成功！
```



## 自定向下的设计

### 局部变量

对于全局变量来说，他会在整个程序存在期间一直存在。

但是有时候，全局变量会让shell函数变得复杂。此时我们需要引入局部变量。

局部变量：

1. 仅仅在定义它们的shell函数中有效，一旦shell函数终止，它们就不复存在。
2. 可以让程序员使用已经存在的变量名称，而不用考虑命名冲突。
3. 通过在变量名前加local来定义的（local x=2）

```shell
#!/bin/bash
#使用局部变量

foo=0 #global varialbe foo

function_1(){
        local foo=1 #local foo
        echo "function_1:$foo"
}

function_2(){
        local foo=2 #local foo
        echo "function_2:$foo"
}

echo "global:foo=$foo"
function_1
echo "global:foo=$foo"
function_2
echo "global:foo=$foo"
```

输出：

```shell
global:foo=0
function_1:1
global:foo=0
function_2:2
global:foo=0
```

## IF分支语句

### 使用If

#### 语法格式：

```shell
if commands; then
			commands
[elif commands; then
			commands...  ]
else
			commands
fi
```

举例：

```shell
x=5
if [$x = 5];then
		echo "x equals 5."
else
		echo "x does not equals 5."
fi
```

输出：

```shell
x does not equals 5.
```

#### 退出状态

命令（包括我们编写的脚本和shell函数）在执行完毕之后，会向操作系统发送一个值，称之为“退出状态”。

这个值是一个0～255之间的整数，用来指示命令执行成功还是失败。0表示执行成功，其他数值表示执行失败。