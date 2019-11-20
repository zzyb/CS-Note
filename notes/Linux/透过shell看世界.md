# 透过shell看世界

## 扩展

​	这里使用`echo` 命令：shell内置命令，把文本参数内容打印到标准输出。

### 路径名扩展

```SHELL
[hadoop@hadoop05 ~]$ ls
apps  bashrc  bin  ccc  --connect  data  ddd  dump.rdb  fff  ggg  hivedata  hivedata2  learn_vim  logs  --password  redis  soft  sparkbin  --username  zookeeper.out
[hadoop@hadoop05 ~]$ echo *
#这里*被扩展为了其他内容，这里扩展成了当前工作目录下符合条件的。
apps bashrc bin ccc --connect data ddd dump.rdb fff ggg hivedata hivedata2 learn_vim logs --password redis soft sparkbin --username zookeeper.out
[hadoop@hadoop05 ~]$ echo ???
#这里？？？被扩展为了其他内容，这里扩展成了当前工作目录下符合条件的。
bin ccc ddd fff ggg
[hadoop@hadoop05 ~]$ echo hello
hello
[hadoop@hadoop05 ~]$ 
```

### 波浪线扩展

- 波浪线字符（～）。
  - 如果用在一个单词开头，将被扩展为指定用户主目录名；
  - 如果没有指定用户命名，则拓展为当前用户的主目录。

```shell
[hadoop@hadoop05 ~]$ echo ~
/home/hadoop
[hadoop@hadoop05 ~]$ echo ~root
/root
[hadoop@hadoop05 ~]$ echo ~tom
~tom
```

### 算数扩展

- `$((expression))`，其中expression指的是包含数值和算数操作符的算术表达式。

| 运算符 | 描述 |
| ------ | ---- |
| +      | 加   |
| -      | 减   |
| *      | 乘   |
| /      | 除   |
| %      | 取余 |
| **     | 取幂 |

```shell
[hadoop@hadoop05 ~]$ echo $((2+2))
4
[hadoop@hadoop05 ~]$ echo $((5%2))
1
```

### 花括号扩展

​	你可以按照花括号里面的模式创建多种文本字符串。

```SHELL
[hadoop@hadoop05 ~]$ echo a{1,2,3}
a1 a2 a3
[hadoop@hadoop05 ~]$ echo number_{1..5}
number_1 number_2 number_3 number_4 number_5
[hadoop@hadoop05 ~]$ echo {a..z}
a b c d e f g h i j k l m n o p q r s t u v w x y z
[hadoop@hadoop05 ~]$ echo {Z..A}
Z Y X W V U T S R Q P O N M L K J I H G F E D C B A
[hadoop@hadoop05 ~]$ echo AA{x{1,2},y{1,2}}AA
AAx1AA AAx2AA AAy1AA AAy2AA
```

- 创建一系列的文件或者目录。

```shell
[hadoop@hadoop05 ~]$ cd ccc
[hadoop@hadoop05 ccc]$ ll
total 0
[hadoop@hadoop05 ccc]$ mkdir {2008..2009}-0{1..9} {2010..2012}-{10..12}
[hadoop@hadoop05 ccc]$ ll
total 108
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-01
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-02
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-03
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-04
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-05
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-06
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-07
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-08
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2008-09
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-01
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-02
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-03
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-04
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-05
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-06
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-07
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-08
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2009-09
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2010-10
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2010-11
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2010-12
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2011-10
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2011-11
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2011-12
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2012-10
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2012-11
drwxrwxr-x 2 hadoop hadoop 4096 Nov 20 11:18 2012-12
```

### 参数扩展

如果存在，则输出变量值，如果打错，没有。

```SHELL
[hadoop@hadoop05 ccc]$ echo $USER
hadoop
[hadoop@hadoop05 ccc]$ echo $SURE

[hadoop@hadoop05 ccc]$ 
```

- 可以使用 `printenv | less` 来查看变量列表！！！

### 命令替换

```shell
[hadoop@hadoop05 ~]$ echo $(which cp)
/bin/cp
[hadoop@hadoop05 ~]$ echo $(ls -l)
total 152 drwxrwxr-x 12 hadoop hadoop 4096 Sep 27 14:40 apps -rw-rw-r-- 1 hadoop hadoop 0 Jul 22 17:03 bashrc drwxrwxr-x 2 hadoop hadoop 4096 Jun 16 2018 bin drwxrwxr-x 29 hadoop hadoop 4096 Nov 20 11:18 ccc -rw-rw-r-- 1 hadoop hadoop 0 Sep 29 16:07 --connect drwxrwxr-x 4 hadoop hadoop 4096 Jun 16 2018 data -rw-rw-r-- 1 hadoop hadoop 76 Sep 3 17:43 dump.rdb drwxrwxr-x 3 hadoop hadoop 4096 Sep 7 19:53 hivedata drwxrwxr-x 3 hadoop hadoop 4096 Sep 25 10:32 hivedata2 drwxrwxr-x 2 hadoop hadoop 4096 Oct 7 15:41 learn_vim drwxrwxr-x 2 hadoop hadoop 4096 Jun 16 2018 logs -rw-rw-r-- 1 hadoop hadoop 321 Sep 29 16:07 --password drwxrwxr-x 3 hadoop hadoop 4096 Sep 4 18:23 redis drwxrwxr-x 2 hadoop hadoop 4096 Jun 16 2018 soft drwxrwxr-x 2 hadoop hadoop 4096 Aug 16 21:50 sparkbin -rw-rw-r-- 1 hadoop hadoop 0 Sep 29 16:07 --username -rw-rw-r-- 1 hadoop hadoop 100294 Nov 19 13:30 zookeeper.out
```

## 引用

### 双引号

- 参数扩展、算数扩展、命令替换 在双引号中依然生效。

- 一旦加上双引号，意味着命令行会被识别为后面只跟着一个参数。

  - 单词分隔机制会把换行符当成界定符。

  - ```shell
    [hadoop@hadoop05 ~]$ echo $(cal)
    November 2019 Su Mo Tu We Th Fr Sa 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
    [hadoop@hadoop05 ~]$ echo "$(cal)"
        November 2019   
    Su Mo Tu We Th Fr Sa
                    1  2
     3  4  5  6  7  8  9
    10 11 12 13 14 15 16
    17 18 19 20 21 22 23
    24 25 26 27 28 29 30
    ```

### 单引号

- 如果我们希望抑制所有的扩展，那么应该使用单引号！！！

```shell
[hadoop@hadoop05 ~]$ echo abc ~/*.out $(echo root) $((2+2)) $USER
abc /home/hadoop/zookeeper.out root 4 hadoop
[hadoop@hadoop05 ~]$ echo "abc ~/*.out $(echo root) $((2+2)) $USER"
abc ~/*.out root 4 hadoop
[hadoop@hadoop05 ~]$ echo 'abc ~/*.out $(echo root) $((2+2)) $USER'
abc ~/*.out $(echo root) $((2+2)) $USER
[hadoop@hadoop05 ~]$ 
```

### 转义字符

- 输出美元符号等
- mv等命令操作中，文件名带有特殊字符。
- 特殊字符包括：
  - $
  - !
  - &
  - 空格

```shell
[hadoop@hadoop05 ~]$ echo i need $500
i need 00
[hadoop@hadoop05 ~]$ echo i need \$500
i need $500
```

