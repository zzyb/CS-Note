# Linux系统

| 命令 | 功能         |
| ---- | ------------ |
| ls   | 列出目录内容 |
| file | 确定文件类型 |
| less | 查看文件内容 |

## ls命令

- 查看目录内容，确定各种重要文件和目录属性。

  - 除了直接显示当前目录外，我们还可以指定要显示的目录。

  - ```shell
    [hadoop@hadoop05 ~]$ ls -l /home/hadoop
    total 152
    drwxrwxr-x 12 hadoop hadoop   4096 Sep 27 14:40 apps
    -rw-rw-r--  1 hadoop hadoop      0 Jul 22 17:03 bashrc
    drwxrwxr-x  2 hadoop hadoop   4096 Jun 16  2018 bin
    drwxrwxr-x  2 hadoop hadoop   4096 Nov 19 09:58 ccc
    -rw-rw-r--  1 hadoop hadoop      0 Sep 29 16:07 --connect
    ```

  - 也可指定多个目录。

  - ```shell
    [hadoop@hadoop05 ~]$ ls -l /home/hadoop/ccc /home
    /home:
    total 4
    drwx------. 19 hadoop hadoop 4096 Nov 19 09:28 hadoop
    
    /home/hadoop/ccc:
    total 0
    -rw-rw-rw- 1 hadoop hadoop 0 Nov 19 09:58 a.txt
    -rw-rw-rw- 1 hadoop hadoop 0 Nov 19 09:58 foo.txt
    ```

  - `-l`选项会长格式显示。

- ls 常用选项
  - | 选项 | 长选项           | 含义                                      |
    | ---- | ---------------- | ----------------------------------------- |
    | -a   | --all            | 列出所有文件（包含点开头的文件）          |
    | -d   | --directory      | 查看目录的详细信息，而不是目录中的内容    |
    | -F   | --classify       | 列出文件的类型指示符（例如，目录会带上/） |
    | -h   | --human-readable | 长格式，以人门可读的方式显示文件大小      |
    | -l   |                  | 长格式显示                                |
    | -r   | --reverse        | 相反顺序显示结果                          |
    | -S   |                  | 按照文件大小排序                          |
    | -t   |                  | 按修改时间排序                            |

## file确定文件类型

```shell
[hadoop@hadoop05 ccc]$ file a.txt 
a.txt: ASCII text
[hadoop@hadoop05 ~]$ file apps
apps: directory
```

## 使用less查看文件内容

- `less filename` 查看文件内容。

  - | 命令        | 功能                                   |
    | ----------- | -------------------------------------- |
    | b           | 向后翻页                               |
    | 空格        | 向前翻页                               |
    | 上箭头      | 向上                                   |
    | 下箭头      | 向下                                   |
    | G           | 文件末尾                               |
    | 1G或者g     | 文件开头                               |
    | /charecters | 向前查找制定字符串                     |
    | n           | 向前查找下一个字符产（之前查找指定的） |
    | q           | 退出                                   |
    | h           | 显示帮助                               |

  

## 符号连接

ll列出文件长格式，符号连接的开头是l，即`lrwxrwxrwx`。