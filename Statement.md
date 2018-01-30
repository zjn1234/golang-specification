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
mark here
