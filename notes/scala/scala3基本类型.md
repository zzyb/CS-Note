# 基本类型与操作

## 基本类型

| 值类型  | 范围 |
| ------- | ---- |
| Byte    |      |
| Short   |      |
| Int     |      |
| Long    |      |
| Char    |      |
| String  |      |
| Float   |      |
| Double  |      |
| Boolean |      |

除了String归于java.lang包外，其余所有的基本类型都是包scala的成员。（如Int的全名是scala.Int）

## 字面量（java跳过）

`字面量`就是写在代码中的常量值。

## 操作符和方法

**任何方法都可以是操作符**

​	到底是操作符还是方法，取决于你如何使用它。如果写成`s.indexOf('o')`indexOf就表示方法；如果写成`s indexOf 'o'`那么indexOf就代表操作符。

## 数学运算（java 跳过）

## 关系与逻辑运算（java 跳过）

## 位操作符（java 跳过）

## 对象相等性

**如果想要比较两个对象是否相等，可以使用`==`，或它的反义`!=`。**

- 这种操作对所有的对象都起作用，不仅仅是基本类型。

```scala
1==2
//false
1!=2
//true
list(1,2,3)==List(1,2,3)
//true
```

- 还可以比较不同类型的两个对象。

```scala
1==1.0
//true
list(1,2,3,)=="hello"
//false
```

- 甚至可以比较null或任何可能是null的东西。不会有任何异常抛出。

```scala
List(1,2,3)==null
//false
null==List(1,2,3)
//false
```

**说明：**

​	遵循简单的规则：首先检查左侧是否为null，如果不是，调用左操作数的equals方法。而精确的比较取决于左操作符的equals方法定义。由于有了自动null检查，因此不需要手动在检查一次了。



**Scala的`==`与Java的有什么差别？**

Java里==既可以比较原始类型也可以比较引用类型。

对于原始类型，java的==比较值的相等性，与Scala一致。

对于引用类型，Java的==比较了引用相等性，也就是说比较的是这两个变量是不是指向JVM中的同一个对象。scala也提供了这种机制，名字是eq。（不过，eq和它的反义词ne仅仅应用于可以直接映射到java的对象）

## 操作符优先级和关联性

## 富包装器

每个基本类型都对应着一个“富包装器”提供许多额外的方法。

**富包装器：**

| 基本类型 | 富包装                 |
| -------- | ---------------------- |
| Byte     | scala.runtime.RichByte |
| Short    | x.x.RichShort          |
| Int      | ...                    |
| Long     | ...                    |
| Char     | ...                    |
| String   | ...                    |
| Float    | ...                    |
| Double   | ...                    |
| Boolean  | ...                    |

**一些富操作：**

| 代码              | 结果                       |
| ----------------- | -------------------------- |
| 0 max 5           | 5                          |
| 0 min 5           | 0                          |
| -2.7 abs          | 2.7                        |
| -2.7 round        | -3L                        |
| 1.5 isInfinity    | false [infinity----无穷大] |
| (1.0/0)isInfinity | true  [infinity----无穷大] |
| 4 to 6            | Range(4,5,6)               |
| "bob" capitalize  | "Bob"                      |
| "abcde" drop 2    | "cde"                      |

