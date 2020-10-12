# 内建控制结构

## if 表达式

- 检测条件是否为真，然后执行两个分支中的一个。

- if能够返回值

  ```scala
  val filename = {
    if(!args.isEmpty)
  		args(0)
    else
    	"default.txt"
  }
  //if表达式返回被选中的值，然后用这个值去初始化filename变量。
  //使用了val，体现了函数式风格，并具有Java的final效果，这样节省了审查变量作用域的所有代码，以检查它是否改变的工作。
  //使用val的第二个好处是，更好的支持等效推论。在表达式没有副作用的前提下，引入的变量等效于计算它的表达式。因此无论何时都可以使用表达式来代替变量名。
  //println(if(!args.isEmpty) args(0) else "default.txt")
  ```

## while循环

- 包括状态判断和循环体，只要状态保持为真，循环体就一遍遍的执行。

  ```scala
  //循环计算最大公约数
  def gcdLoop(x:Long,y:Long):Long={
    var a = x
    var b = y
    while(a!=0){
      val temp = a
      a = b%a
      b = temp
    }
    b
  }
  ```


- do-while循环

  与while基本没有什么区别，只是把状态检查移到了循环体之后。

  ```scala
  var line = ""
  do{
    line = readLine()
    println("Read:"+line)
  }while(line!="")
  //利用do-while从标准输入读取信息
  ```

- **while和do-while结构是“循环”而不是表达式！因为它们不能产生有意义的结果，结果类型是`Unit`**，表明存在并且唯一存在类型为Unit的值，称之为unit value，写成()。

  - `()`的存在是Scala的Unit不同于Java的void的地方。

  - ```scala
    scala> def greet() {println("hi")}
    greet: ()Unit
    //定义一个函数，由于方法体之前没有等号，greet被定义为结果类型为Unit的过程。因此greet返回unit值，()。
    Scala> greet() == ()
    hi
    res0: Boolean = true
    //比较greet()返回的结果与(),得到true。
    ```

- 慎重使用while（除非非用不可，不然考虑其他方式）

## For表达式

### 枚举集合类

​	for能做的最简单的事就是把集合中的元素都枚举一遍。

```scala
//枚举当前目录中的文件
val filesHere = (new java.io.File(".")).listFiles
for(file <- filesHere)
	println(file)

//在其他语言中，我们常常采用如下方式。但是这在scala中不常见。
//不常见的原因是：集合对象本身可以直接被枚举。
for(i <- 0 to filesHere.length-1)
	println(filesHere(i))
```

- for表达式对任何种类的集合都有效，不只是数组。

```scala
//枚举Range类型
//使用1 to X 来创建Range，然后使用for做枚举。
for(i <- 1 to 4)
	println("Iteration"+i)
```

如果不想枚举Range的上界，可以使用until代替to：

```scala
for(i <- 1 until 4)
	println("Iteration"+i)
//会输出结果1、2、3.
```

### 过滤

​	有时候，你可以只过滤某个子集合。通过在for表达式中添加过滤器实现(filter)，即if语句。

```scala
val filesHere = (new java.io.File(".")).listFiles
for(file <- filesHere if file.getName.endsWith(".scala"))
	println(file)
//使用带过滤器的for发现.scala文件
```

- 如果需要的话，我们可以包含更多的过滤器。只需要不断增加if子句即可。
  - 如果发生器中加入**超过一个过滤器，if子句必须用分号分隔。**

```scala
val filesHere = (new java.io.File(".")).listFiles
for(
	file <- filesHere
  if file.isFile;
  //注意这里的if语句最后带一个分号！
  if file.getName.endsWith(".scala")
)	println(file)
//仅仅打印输出文件，而不是目录。
```

### 嵌套枚举

如果加入多个 `<- 子句`，就可以得到嵌套循环。

```scala
def fileLines(file:java.io.File) = scala.io.Source.fromFile(file).getLines.toList

def grep(pattern:String) = 
	for(
  file <- filesHere
    if file.getName.endsWith(".scala")
  line <- fileLines(file)
    if line.trim.matches(pattern)
    //对line.trim第一次计算（下面优化）
  ) println(file + ":" + line.trim)
//对line.trim第二次计算（下面优化）
//外层循环枚举filesHere，内层枚举所有以.scala结尾文件的fileLines(file)

grep(".*gcd.*")
```

### 流间（mid-stream）变量绑定

​	for循环中有时会存在相同的多次计算，我们可以在for循环中使用等号(=)把一次计算结果绑定到新的变量。绑定的变量被当作val引入和使用，不过不带val关键字。

```scala
def grep(pattern:String) = 
	for{
    file <- filesHere
    if file.getName.endsWith(".scala")
    line <- fileLines(file)
    trimmed = line.trim
    //这里对fileLines(file)得到的.scala文件去除空格，并保存到trimmed中
    if trimmed.matches(pattern)
    //第一次使用trimmed
  }	println(file + ":" +trimmed)
//第二次使用trimmed（只计算了一次）

//trimmed变量被从半路引入for表达式，并被初始化为line.trim的结果值。并在if语句、println中使用了两次。
grep(".*gcd.*")
```

### 制造新集合

​	对于for循环，我们常常只是对枚举值进行操作之后释放；除此之外，我们可以创建一个值，去记住每一次迭代，只要在for表达式之后加上关键字`yield`。

```scala
def scalaFiles = 
	for{
    file <- filesHere
    if file.getName.endsWith(".scala")
  } yield file
```

- 注意放置yield关键字的地方。

```scala
for {子句} yield {循环体}
```

## 使用try表达式处理异常

### 抛出异常

创建异常对象，然后trow关键字抛出：

```scala
throw new IllegalArgumentException
```

- 在scala中，throw也是有结果类型的表达式。

```scala
val half = 
	if(n%2==0)
		n/2
	else
		throw new RuntimeException("n must be even")
//如果n为偶数，half将被初始化为n的一半，如果不是偶数，异常将在half初始化之前被抛出。
```

### 捕获异常

- 首先执行程序体，如果抛出异常，则依次尝试每个catch子句；如果是第一个异常，则第一个子句被执行，如果是第二个异常，则第二个子句被执行，如果都不是，那么try-catch将被终结并把异常上升出去。
- 与Java的不同是，Scala中不需要捕获检查异常，或者把它们声明在throws子句中。（如果愿意，可以使用@throws注解声明，但这不是必须的。）

```scala
try{
  val f = new FileReader("input.txt")
}catch{
  case ex: FileNotFoundException => //处理丢失的文件
  case ex: IOException => //处理I/O错误
}
```

### finally子句

让某些代码无论方法如何终止都要执行，可以把表达式放入finally子句中。

```scala
val file = new FileReader("input.txt")
try{
  //使用文件
}finally{
  file.close() //确保关闭文件
}
```

- 通常finally子句做一些诸如关闭文件之类的清理工作。

### 生成值

try-catch-finally能够生成值。

```scala
def urlFor(path:String) = 
	try{
    new URL(path)
  }catch{
    case e: MalformedURLException => new URL("http://www.baidu.com")
  }
//catch子句产生值
```

- 避免用finally子句返回值，而是用来确保某些操作发生的途径。

## 匹配表达式

Scala的match表达式类似于其他语言中的switch语句，提供给你多个备选项中做选择。

```scala
val firstArg = if(args.length>=0) args(0) else ""

firstArg match {
  case "Java" => println("面向对象编程")
  case "shell" => println("脚本语言")
  case "hadoop" => println("大数据")
  case _ => println("无")
}
//有副作用的match表达式
```

与Java的switch语句的差别：

- 任何类型的常量，或其他什么东西，都可以当作Scala里做比较的样本(case)，而不仅仅是java中的整数类型和枚举类型。
- 备选项后面没有break。（break是隐含的。）
- match 表达式能够产生值。

```scala
val firstArg = if(args.length>=0) args(0) else ""

val friend =
  firstArg match {
    case "Java" => "面向对象编程"
    case "shell" => "脚本语言"
    case "hadoop" => "大数据"
    case _ => "无"
  }
println(friend)
//match表达式生成返回值并存储在friend变量里。
```

## 不在使用break和continue

## 变量范围

注意

- Java不允许内部范围内创建与外部范围变量同名的变量。但在scala中可以，因为scala中内部变量被认为遮蔽了同名的外部变量。

## 重构指令式代码风格