# Statements

语句控制执行

结束语句为以下其中之一：
1. return或者是goto语句
2. 调用内建函数panic
3. 在一个代码块内，语句列表以一个结束语句而结束
4. 带有else的if语句，if或者else分支都是结束语句
5. 不带break的for语句，并且没有循环条件（死循环）
6. 不带break的switch语句，每个分支（包括default分支）的语句列表以一个结束语句而结束，或者是一个标记为fallthrough的语句
7. 不带break的select语句，每一个select分支的语句列表以一 个结束语句而结束
8. 一个标记为结束语句的语句

除以上外其他所有语句为非结束语句

上面说的语句列表以一个结束语句而结束的意思是列表非空，并且由最后一条非空语句结束

## Empty statements
空语句不做任何事情

## Labeled statements
被标记的语句很可能是goto、break、continue语句要跳转的目标，
例如：`Error: log.Panic("error encountered)`

## Expression statements
除了特殊的某些内建函数外，其它的函数、方法调用和channel的接收操作可以作为语句出现，这样的语句可以被括号括起来。

下面的内建函数不允许作为语句单独出现：
`append cap complex imag len make new real unsafe.Alignof unsafe.Offsetof unsafe.Sizeof`
例如：
```
h(x+y) 合法
f.Close()合法
<-ch 合法
(<-ch)合法
len("foo")如果len是内建函数，那这样使用就是非法的 x := len("foo")这样是合法的
```

## Send statements
发送语句向channel中写入一个值。channel的表达式必须是对应channel的类型，channel指示方向必须是允许发送的操作，并且要写入的值的类型必须是对应channel的元素类型
```
SendStmt = Channel "<-" Expression .
channel = Expression .
```
channel和值的表达式是在与channel通信前被执行的。与channel的通信会被阻塞到发送可以进行的时候。 对于非缓存channel而言，如果接收者的状态是ready，则对该channel的发送操作可以被立刻处理。对于缓存的channel而言，如果该缓存channel中还有空间，则对该channel的发送操作可以被立刻处理。向一个已经被关闭的channel执行写入操作会导致运行时panic。向一个nil channel执行写入请求会被永远阻塞。

`ch <- 3 向channel ch 中写入数字3`

## IncDec statements

"++"和"--"语句会给他们的操作数增加或者减少1。和赋值一样，操作数必须是可寻址的或者是是一个map索引表达式

`IncDecStmt = Expression ( "++" | "--" ) .`

下面的赋值语句从语义上来讲是等同的：
```
IncDec statement          Assignment
x++							x += 1
x--							x -= 1
```

## Assignments
```
Assignment = ExpressionList assign_op ExpressionList .
assign_op = [ add_op | mul_op ] "=" .
```

每一个左操作数必须是可寻址的，又或者是一个map索引表达式，或者（仅仅是对赋值而言）是 _. 操作数可以被括号括起来。
```
x = 1
*p = f()
a[i] = 23
(k) = <-ch 和 k = <-ch是一样的
```

在一个x op= y的赋值操作中，op是一个任意的数学操作符，这种赋值操作的方法和x = x op (y)是相等的，但是x op= y只将x计算一次。op=结构是一个单独的符号。在赋值操作中，左右表达式列表必须只包含一个单值表达式，并且左表达式不能是空白标识符（_）
```
a[i] <<= 2
i &^= 1 << n
```

元组赋值将单个表达式的多个返回值赋给变量列表。有两种形式。第一种形式，右表达式是个有多返回值的单个表达式例如函数调用，channel或者map操作，或者是类型断言。左操作数的数量必须和上述返回值的数量相匹配。例如，如果一个函数f返回两个值的话，那
`x, y = f()`

将f()的第一值赋给x，第二个值赋给y。第二种形式，左边操作数的数量必须等于右边操作数的数量，这些操作数中的每一个必须是单值的，并且右边第n个表达式会被赋值给左边第n个表达式，例如：
`one, tow, three = "一", "二", "三"`

空白标识符提供了一个在赋值语句中忽略右值的办法，例如：
```
_ = x        // 计算x但是忽略x的值
x, _ = f()   // 计算f() 但是忽略f()的第二个返回值
```

赋值分两个阶段进行。首先，左边的索引表达式、指针间接引用操作数和右边的表达式全部按照文档中提到的the usual order顺序来计算。然后，按照从左到右的顺序进行赋值操作。

```
a, b = b, a  // 交换a和b的值

x := []int{1, 2, 3}
i := 0
i, x[i] = 1, 2  // 设 i = 1, x[0] = 2

i = 0
x[i], i = 2, 1  // 设 x[0] = 2, i = 1

x[0], x[0] = 1, 2  // 设 x[0] = 1, 然后 x[0] = 2 (按上面的顺序来看 x[0] == 2 是最后被赋值的)

x[1], x[3] = 4, 5  // 先将x[1]设为1，然后将x[3]设为4的时候panic.

type Point struct { x, y int }
var p *Point
x[2], p.x = 6, 7  // 先将x[2] 设为6, 然后设p.x = 7的时候panic，因为p未被分配空间

i = 2
x = []int{3, 5, 7}
for i, x[i] = range x {  // 先设置i = 0 然后x[0] = 3
	break
}
循环结束后，i = 0 x = []int{3, 5, 3}
```
在赋值语句中，每一个值必须被赋值给和它相同类型的操作数，并且有下列特殊情况：
 1. 任意类型的值都可以被赋值给空白标识符
 2. 如果一个非类型化的常量被赋值给一个接口类型的变量或者是一个空白标识符，该常量会先转换成它默认的类型。
 3. 如果一个非类型化的布尔类型值被赋值给一个接口类型的变量或者一个空白标识符，该值先被转换成bool类型

## If statements
"if" 语句根据布尔表达式的值来确定两个分支的执行。如果表达式值为真，"if"分支执行，否则 如果 "else"存在的话，则执行else分支。

`IfStmt = "if" [ SimpleStmt ";" ] Expression Block [ "else" ( IfStmt | Block ) ] .`

```
if x > max {
	x = max
}
```

在执行if表达式前可以先执行一个简单的表达式，例如：
```
if x := f(); x < y {
	return x
} else if x > z {
	return z
} else {
	return y
}
```

## Switch statements
"Switch" 语句提供了多个执行路径。在"Switch"中，表达式或者类型说明符会和"Cases"做比较，从而决定执行哪一个分支

`SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .`

有两种形式：表达式"Switch"和类型"Switch". 在表达式"Switch"中，"Case"包含的表达式会和"Switch"包含的表达式值作比较。在类型"Switch"中，包含类型的"Case"会和"Switch"的类型表达式作比较。"Switch"表达式在一个"Switch"语句中只被计算一次。

### Expression switches
在一个表达式"Switch"中，"Switch"表达式先被计算，然后是"Case"表达式，"Case"的表达式值不需要是常量，计算的顺序是从左到右，从上到下;第一个匹配"Switch表达式值的"Case"的语句会被立刻执行;其它的"Case"语句会被直接跳过。如果没有"Case"表达式能匹配的上并且同时存在有"default case"，则该语句会被执行。如果一个Switch语句不包含任何表达式，则默认表达式为布尔类型的true值。

```
ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ] "{" { ExprCaseClause } "}" .
ExprCaseClause = ExprSwitchCase ":" StatementList .
ExprSwitchCase = "case" ExpressionList | "default" .
```
如果"Switch"表达式计算结果是一个非类型化的常量，那么首先这个值会被转化成该值对应的默认类型;如果这个值是一个非类型化的布尔值，那么这个值会被转化成bool类型。预声明的非类型化值nil不能被用在"Switch"表达式中。

如果一个"Case"表达式的结果是非类型化的，这个结果会先被转化成"Switch"表达式结果的类型。对于每一个"Case"表达式x和"Switch"表达式的值t，x == t 必须是一个有效的比较

换句话说，"Switch"表达式就是用来声明并初始化一个临时的没有显示类型的变量，把这个变量比较上面说的t，然后用这个t用来和上面说到的"Case"表达式x作是否相等的比较测试

在一个"Case"或者一个"Default Case"中，最后一个非空语句可以是一个(可能是被标记过的)"fallthrough"语句来表示控制流应该从这个子句的末尾流向下一个子句的第一条语句。否则控制流将会流向"Switch"语句的末尾。"fallthrough"语句可以是所有语句的最的一条语句，唯独不能是"Switch"表达式的最后一条Case子句内。

"Switch"表达式之前可以写一简单的表达式，该表达式在"Switch"之前被执行

```
switch tag {
	default: s3()
    case 0, 1, 2, 3: s1()
    case 4, 5, 6, 7: s2()
}

switch x := f(); {
	case x < 0: return -x
    default: return x
}

switch {
	case x < y: f1()
    case x < z: f2()
    case x == 4: f3()
}
```

实现约束：编译器有可能会不允许多个"Case"有相同常量值。例如，在"Case"表达式中，当前的编译器不允许出现重复的整数、浮点数或者字符串常量。

## Type switches
类型"Switch"比较的是类型而不是常量。类型"Switch"在其它地方和表达式"Switch"是相似的。类型"Switch"由一个特殊的"Switch"表达式组成，这个表达式用保留字type来作类型断言，而不是使用实际的类型。
```
switch x.(type) {
	// cases
}
```

然后，"Cases"语句会使用实际的类型T和表达式x的动态类型作匹配。正如类型断言中所述，x必须是一个接口类型，并且每一个"Case"中的非接口类型T必须实现的x的类型。每一个"Case"中的类型相互之间必须是不同的。

```
TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard "{" { TypeCaseClause } "}" .
TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
TypeCaseClause  = TypeSwitchCase ":" StatementList .
TypeSwitchCase  = "case" TypeList | "default" .
TypeList        = Type { "," Type } .
```

上面说到的TypeSwitchGuard可以包括一个简单的变量声明，当这种形式被用于其中的时候，该变量会被声明在每一个"Case"子句的区块末尾。在每一个只有一个类型的"Case"子句中，该变量即为该Case中的类型;否则，该变量的类型为TypeSwitchGuard中的表达式类型。

在"Case"中的类有可能会是nil;该"Case"用于当TypeSwitchGuard中的表达式是一个nil接口值的时候。至多只能有一个nil "Case"

给予一个interface{}类型的表达式x，看下下面的例子：
```
switch i := x.(type) {
case nil:
	printString("x is nil")                // type of i is type of x (interface{})
case int:
	printInt(i)                            // type of i is int
case float64:
	printFloat64(i)                        // type of i is float64
case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
default:
	printString("don't know the type")     // type of i is type of x (interface{})
}
```

上面的内容可以被重写为：
```
v := x  // x is evaluated exactly once
if v == nil {
	i := v                                 // type of i is type of x (interface{})
	printString("x is nil")
} else if i, isInt := v.(int); isInt {
	printInt(i)                            // type of i is int
} else if i, isFloat64 := v.(float64); isFloat64 {
	printFloat64(i)                        // type of i is float64
} else if i, isFunc := v.(func(int) float64); isFunc {
	printFunction(i)                       // type of i is func(int) float64
} else {
	_, isBool := v.(bool)
	_, isString := v.(string)
	if isBool || isString {
		i := v                         // type of i is type of x (interface{})
		printString("type is bool or string")
	} else {
		i := v                         // type of i is type of x (interface{})
		printString("don't know the type")
	}
}
```

TypeSwitchGuard之前可以添加一个简单的语句，该语句在TypeSwitchGuard之前被执行。
"fallthrough"语句在类型"Switch"中是不允许使用的。

## For statement
"For"语句表示某一区块的重复性执行动作。有三种形式：迭代操作可以被单个条件控制，该条件可能是一个"for"子句，也可以是一个"range"子句。
```
ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
Condition = Expression .
```

### For statements with single condition
在这种简单形式中，只要该布尔条件的计算结果为true，刚该"for"语句会确定区块重复执行与否。条件的计算是发生在每次迭代操作之前的。如果该条件表达式被省略，则默认表示为true。

```
for a < b {
	a *= 2
}
```

### For statements with clause
拥有For子句的"For"语句也是受它的条件所控制的，除此之外，该"for"语句还可以包含一个前置和后置语句，就好比赋值、加语句或者减语句。前置语句可以是一个简单的变量声明，但是后置语句一定不能是变量声明。被前置语句声明的变量被重复用于每一次的迭代过程中。

```
ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
InitStmt = SimpleStmt .
PostStmt = SimpleStmt .
```

```
for i := 0; i < 10; i++ {
	f(i)
}
```

### For statement with range clause

带有"range"子句的"for"语句会迭代array、slice、string、map、接收自channel的所有项。每一项中的迭代值被赋给对应的迭代变量，然后才执行代码块。

`RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .`

"range"子句中的右侧表达式叫作range 表达式， 其有可能是数组或者是指向数组的指针，slice，string，map，或者是允许接收操作的channel。正如赋值语句一样，如果存在左操作数的话，则它必须是可寻址的或者是map的索引表达式;这些左操作数代表了迭代变量。如果range表达式是一个channel，刚最多允许一个迭代变量的存在，否则最多允许存在两个。如果最后一个迭代变量是空白标识符，这样的range子句就等于没有标识符的子句。

range表达式会在循环开始前被计算一次，但是有一个特例，如果range表达式是一个数组或者一个指向数组的指针并且最多存在一个迭代变量，则在循环前只有range表达式的长度才会被计算;如果长度是一个常量值，（by definition）range表达式本身是不会被计算的。

左侧的函数调用每一个迭代只会执行一次。对于每一次迭代而言，迭代值的产生是由于对应的迭代变量的存在。

```
Range expression                          1st value          2nd value

array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
string          s  string type            index    i  int    see below  rune
map             m  map[K]V                key      k  K      m[k]       V
channel         c  chan E, <-chan E       element  e  E
```
1. 对于一个数组，指向数组的指针，或者是slice值a而言，迭代的索引值是按照递增的顺序出现的，并且以0为起始。如果最多只有一个迭代变量存在，range会循环产生从0到len(a)-1的数值，并且不会索引进数组或者slice本身的值。对于一个nil slice而言，迭代次数为0.
2. 对于一个字符串值而言，"range"子句会从代表字符串第一个unicode字符索引为0的地方开始迭代，迭代成功后，第一个迭代变量即是utf-8编码字符的第一个字节的索引值，第二个迭代变量即是该迭代对应的utf-8编码值，类型为rune。如果迭代过程遇到了一个无效的utf-8序列值，第二个迭代变量的值为0xFFFD，代表unicode置换字符，下一次迭代将会往前移动一个字节。
3. map的迭代顺序是不确定的，也不能保证每一每一次的迭代顺序是一致的。如果一个map项内容在还没有被迭代到的时候被删除了，则对应的迭代值就不变再出现。如果在迭代的过程中创建了一个新的map项，刚这个新创建的map项有可能会被迭代，有可能会被跳过。迭代过程中，第一个被创建的map项，从上次迭代到下次的顺序是不可选择的。如果map是nil，刚迭代的次数为0.
4. 对于channel而言，产生的迭代值就是那些被成功被写入channel的值，该迭代会持续到该channel被关闭。如果channel为nil，则迭代过程会永久堵塞。

则如赋值操作一样，迭代值会被赋给迭代变量。

迭代变量可以在"range"子句中以短变量声明（:=）的方式被声明。在这种情况下，变量的类型被设置为对应的迭代值的类型并且他们的作用范围是整个的"for"语句;这些变量会在每一次迭代中被重复使用。如果迭代变量被声明在了"for"语句的外面，在迭代执行完后，该变量的值为最后一次的迭代值。

```
var testdata *struct {
	a *[7]int
}
for i, _ := range testdata.a {
	// testdata.a is never evaluated; len(testdata.a) is constant
	// i ranges from 0 to 6
	f(i)
}

var a [10]string
for i, s := range a {
	// type of i is int
	// type of s is string
	// s == a[i]
	g(i, s)
}

var key string
var val interface {}  // value type of m is assignable to val
m := map[string]int{"mon":0, "tue":1, "wed":2, "thu":3, "fri":4, "sat":5, "sun":6}
for key, val = range m {
	h(key, val)
}
// key == last map key encountered in iteration
// val == map[key]

var ch chan Work = producer()
for w := range ch {
	doWork(w)
}

// empty a channel
for range ch {}
```

## GO statement
在同一地址空间内，go语句将一函数调用当作独立的goroutine来运行。

`GoStmt = "go" Expression .`

go语句的表达式必须是一个函数或者方法调用;并且不能被括号括起来。内建函数的调用正如[expression statements]()中所限制的一样。

在调用的goroutine中，函数值和参数像正常调用函数那样被计算，但是它又不像一个正常的函数调用，程序的执行并不会等待该函数的结束。相反，该函数会在一个新的goroutine中独立执行。当函数结束的时候，它的goroutine也随之结束。如果该函数有任何的返回值，当函数结束的时候，这些返回值会被全部忽略。

```
go Server()
go func(ch chan<- bool) { for { sleep(10); ch <- true }} (c)
```

## Select statements

"select"语句会选择一组可以发送或者接收的操作。它看起来很像"switch"语句，但是"select"语句的所有case全部是针对传输操作的。

```
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( SendStmt | RecvStmt ) | "default" .
RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
RecvExpr   = Expression .
```

RecvStmt会将RecvExpr的结果赋值给一个或者两个变量，这些变量可以使用简单声明语句来声明。RecvExpr必须是一个（可以被括号括起来）接收操作。"select"语句中可以最多出现一个default Case并且它可以发现在case列表的任何地方。

"select"语句的执行分下面几个步骤
 1. 对于所有的case语句而言，一旦进入到"select"语句内，接收操作的channel操作数和channel还有右边的发送语句表达式只会被计算一次，并且是以源代码的顺序。结果就是一系列要发送和接收的channels，还有相对应的要发送的值。无论选择哪一种传输操作，都有可能发生任意的边际影响。此时，在RecvStmt内的带有短声明或者赋值的左表达式还没有被计算。
 2. 如果其中存在多个可以处理的操作，通过一个统一的伪随机选择方法，只能有一个case被选择执行。否则的话，如果存在default case，则会执行该子句。如果没有default case，"select"语句会一直堵塞直到其中的一个case语句可以运行。
 3. 除非选择的是default case，否则相应的case语句会被执行。
 4. 如果被选中的case语句是一个带短声明或者赋值操作的RecvStmt，则左表达式会被计算，并且接收值会被赋给左表达式。
 5. 最后是被选中的"select"的case的语句列表被执行。

因为发生在nil channel上的通信永远不会被执行，因此，一个带有nil channel并且没有default case的"select"语句会被永远堵塞。

```
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // same as: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c
	select {
	case c <- 0:  // note: no statement, no fallthrough, no folding of cases
	case c <- 1:
	}
}

select {}  // block forever
```

## Return statements

在一个函数F中的return语句会终止函数F的执行，并且可选择性的提供一个或者多个结果值。任何被函数F defer的函数会在F返回到它的调用者之前被执行。

`ReturnStmt = "return" [ ExpressionList ] .`

在一个没有返回类型的函数中，return语句不能跟任何的返回值。
```
func noResult() {

}
```

有三种方法可以从一个带返回类型的函数中返回值：
 1. 返回的单个或者多个值可以被显示的在return语句中列出来。每一个返回值的表达式必须是单值的并且是可以被赋值给对应类型的函数返回值元素。
 2. return语句中的表达式列表可以是一个返回多值的函数调用。结果就好像从该函数返回的每一个值都被赋值给相应类型的临时变量，紧接着跟着一个return语句，将这些临时变量值返回，在这一点上，之前的case规则也同样适用。
 3. 如果函数的返回结果类型确定了参数名字，return语句的表达式列表可以是空的。结果参数和普通的变量是一样的，并且函数必要的时候也可以赋值给他们。return语句会返回这些变量的值。

无论如何声明，一旦进入到函数内部，所有的结果值全部会被初始化成对应类型的0值。在任何被deferred的函数被执行前，return语句会先将值传递给结果参数。

实现约束：如果一个和返回结果相同名字的不同实体存在于return位置的范围内时，这时候编译器一般不允许return语句中出现空表达式列表。例如：
```
func f(n int) (res int, err error) {
	if _, err := f(n-1); err != nil {
		return  // invalid return statement: err is shadowed
	}
	return
}
```

## Break statements

break语句会终止带有相同功能的最深层的"for"，"switch","select"语句的执行。

`BreakStmt = "break" [ label ] .`

如果存在一个标签，必须是一个封闭的"for","switch",或者是"select"语句，这才是执行终止的地方

## Continue statements

continue语句从最内层的for循环的后置语句开始下一次迭代。for循环必须在相同的函数内。

`ContinueStmt = "continue" [ Label ] .`

如果存在一个标签，必须是一个封闭的"for"语句，这才是下一次执行开始的地方。
```
RowLoop:
	for y, row := range rows {
		for x, data := range row {
			if data == endOfRow {
				continue RowLoop
			}
			row[x] = data + bias(x, y)
		}
	}
```

## Goto statements

goto语句会将控制权转移到相同函数内对应的标签语句上。

`GotoStmt = "goto" label .`

`goto Error`

执行goto语句的时候千万不要导致任何不在goto范围内的变量进入到goto的范围内，例如下面的这个例子：
```
	goto L   //bad goto
    v := 3
L:
```
上面的goto用法是错误的，因为它跳过了v变量的创建。

在代码块外的goto语句不能跳到代码块内的标签。例如以下这个例子：
```
if n%2 == 1 {
	goto L1
}
for n > 0 {
	f()
	n--
L1:
	f()
	n--
}
```

上面的goto用法是不对的，因为标签L1在for语句的代码块内部，但是goto不在其内部，所以无法跳转。

## Falthrough statements

fallthrough 语句将控制权限转移到switch语句中的下一个case语句的第一条语句。他可以作为这样的case子句中的最后一条非空语句。

`FallthroughStmt = "fallthrough" .`

## Defer statements
defer语句会调用一个函数而该函数的执行发生在调用defer的函数返回后，或者是该函数执行了一个return语句，或者是到达函数的末尾，或者是因为对应的goroutin panic了

`DeferStmt = "defer" Expression .`

defer的表达式必须是一个函数或者方法;并且不能被括号括起来，调用内建函数的约束如[expression statements]()

每次当defer语句执行的时候，被调用函数的值和参数会像正常规则那样被计算，并且保存一份新的内容，但是实际的函数调用还未发生。相反，被defer的函数在调用defer的函数结束后会被立刻调用，并且以函数被defer的相反顺序。如果一个被defer的函数的值是nil，当函数被真正调用的时候，程序会panic，defer语句执行的时候并不会panic。

例如，如果一个被defer的函数是一个[function literal]()(匿名函数)并且调用他的函数拥有被命名的结果参数在该匿名函数的范围内，则被defer的函数可以在这些参数被返回前访问和修改他们。如果被defer的函数有任何返回值，刚当该函数执行完后，这些值会被忽略。

```
lock(l)
defer unlock(l)  // unlocking happens before surrounding function returns

// prints 3 2 1 0 before surrounding function returns
for i := 0; i <= 3; i++ {
	defer fmt.Print(i)
}

// f returns 1
func f() (result int) {
	defer func() {
		result++
	}()
	return 0
}
```