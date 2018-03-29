# Expressions
表达式通过将操作符和函数应用于操作数来表示值的计算。

## Operands
操作数代表表达式中的基本值。一个操作数可以是字面值，一个非下划线标识符的常量 ，变量，函数，或者括号表达式。

下划线标识符只能出现在赋值语句的左操作数中。
```
Operand     = Literal | OperandName | "(" Expression ")" .
Literal     = BasicLit | CompositeLit | FunctionLit .
BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
OperandName = identifier | QualifiedIdent.
```

## Qualified identifiers
合法的标识符是指一个带有包名前缀的标识符。包名和前缀都不能为空。

`QualifiedIdent = PackageName "." identifier .`

一个合法的标识符会访问其它包中的标识符，该包必须被导入，而且要访问的标识符必须是可导出的，并且声明在了包级别的代码块中。

`math.Sin	// denotes the Sin function in package math`

## Composite literals
复合字面值会为structs，arrays，slices，和maps这些复合类型构造值，并且会在每次复合类型被计算的时候创建新的值。复合字面值由字面值类型组成，并且后跟大括号括起来的元素列表。每一个元素前可以添加相对应的key值。

```
CompositeLit  = LiteralType LiteralValue .
LiteralType   = StructType | ArrayType | "[" "..." "]" ElementType |
                SliceType | MapType | TypeName .
LiteralValue  = "{" [ ElementList [ "," ] ] "}" .
ElementList   = KeyedElement { "," KeyedElement } .
KeyedElement  = [ Key ":" ] Element .
Key           = FieldName | Expression | LiteralValue .
FieldName     = identifier .
Element       = Expression | LiteralValue .

```

复合字面类型的底层类型必须是struct，array，slice或者map（除Type别名外，编译器会强制检测该约束）。元素和key的类型必须可以被赋值给对应的域的元素和key的字面值类型;这其中没有其它任何类型的转换。对于struct，array，slice和map而言，key会分别被解释成字段的名字，索引，索引和key。对于map字面值而言，每一个元素都要有对应的key。以相同的字段名字或者是常量key值来指定多个元素的行为是不正确的。对于非常量的map的key而言，请参考[evaluation order]()

以下规则适用于struct字面类型：
 - 声明在struct类型中的key必须是一个字段的名字。
 - 一个不包括任何key的元素列表必须以struct的字段声明的顺序来依次列出每一个元素。
 - 只能其中一个元素有key，则其它的元素也必须有key。
 - 包含key的元素列表不需要列出该struct所有字段的元素值。省略的字段会被初始化为该字段类型对应的0值。
 - struct字面类型可以省略元素列表，这样的字面类型值会初始化为该类型对应的0值。
 - 访问一个其它包中的struct的非导出字段元素是错误的。

给予以下声明：

```
type Point3D struct { x, y, z float64 }
type Line struct { p, q Point3D }
```

可以有如下写法：

```
origin := Point3D{}                            // zero value for Point3D
line := Line{origin, Point3D{y: -4, z: 12.3}}  // zero value for line.q.x
```

以下规则适用于array和slice字面类型：
 - 第一个元素都绑定了一个整形的索引值，来表示该元素在数组中的位置。
 - 一个有key的元素会使用该key作为索引。该key必须是一个非负常量值，并且可以一个int类型的值所表示;如果该key有类型，则该类型一定是整数类型。
 - 没有指定key的元素使用前一个元素的索引加1来表示该元素的索引。如果第一个元素没有key，则该索引为0。

取复合字面的地址会生成一个指针，该指针指向一个唯一变量，该变量由该复合字面值初始化。

`var pointer *Point3D = &Point3D{y: 1000}`

数组字面值的长度是该字面类型所确定的长度。如果元素列表中提供的元素个数少于字面类型所指定的长度，余下的元素会被初始化成对应元素类型的0值。元素列表中提供的元素大于字面类型所指定的长度是错误的。符号...表示数组的长度等于最大的元素索引值加1。

```
buffer := [10]string{}             // len(buffer) == 10
intSet := [6]int{1, 2, 3, 5}       // len(intSet) == 6
days := [...]string{"Sat", "Sun"}  // len(days) == 2
```

slice字面值表示整个的底层数组字面值。因此slice的长度和能力是最大的元素索引值加1。slice字面值有以下形式

`[]T{x1, x2, … xn}`

也可以被简写成对数组的切片操作:

```
tmp := [n]T{x1, x2, … xn}
tmp[0 : n]
```

在一个T类型的array，slice或者map的复合字面值中，本身就是复合类型的元素或者map的key如果和T类型的key或者元素完全相同，则这些复合类型的元素或者map的key是可以省略的。相似的，当元素或者Key的类型为*T时，为复合字面值地址的元素或者key可以省略&T
。

```
[...]Point{{1.5, -3.5}, {0, 0}}     // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}
[][]int{{1, 2, 3}, {4, 5}}          // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}
[][]Point{{{0, 1}, {1, 2}}}         // same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}
map[string]Point{"orig": {0, 0}}    // same as map[string]Point{"orig": Point{0, 0}}
map[Point]string{{0, 0}: "orig"}    // same as map[Point]string{Point{0, 0}: "orig"}

type PPoint *Point
[2]*Point{{1.5, -3.5}, {}}          // same as [2]*Point{&Point{1.5, -3.5}, &Point{}}
[2]PPoint{{1.5, -3.5}, {}}          // same as [2]PPoint{PPoint(&Point{1.5, -3.5}), PPoint(&Point{})}
```

当一个使用type别名形式的复合字面值以操作数的形式出现在关键字[keyword]()和if，for，或者switch语句代码块的左大括号中间时，会发现解析歧义，此时，该复合字面值不会被闭合在小括号，方括号或者大括号中。在这种情况下，复合字面值的左大括号被错误的解析成下文代码块的开始大括号。为了解决该歧义，该组合字面值必须被括号括起来。

如下所示：
```
if x == (T{a,b,c}[i]) { … }
if (x == T{a,b,c}[i]) { … }
```

有效的array，slice和map字面值的例子如下：

```
// list of prime numbers
primes := []int{2, 3, 5, 7, 9, 2147483647}

// vowels[ch] is true if ch is a vowel
vowels := [128]bool{'a': true, 'e': true, 'i': true, 'o': true, 'u': true, 'y': true}

// the array [10]float32{-1, 0, 0, 0, -0.1, -0.1, 0, 0, 0, -1}
filter := [10]float32{-1, 4: -0.1, -0.1, 9: -1}

// frequencies in Hz for equal-tempered scale (A4 = 440Hz)
noteFrequency := map[string]float32{
	"C0": 16.35, "D0": 18.35, "E0": 20.60, "F0": 21.83,
	"G0": 24.50, "A0": 27.50, "B0": 30.87,
}
```

## Function literals
函数字面值代表了匿名函数。

`FunctionLit = "func Signature FunctionBody .`

`func(a, b int, z float64) bool {return a*b < int(z)}`

函数字面值可以被赋值给一个变量，也可以被直接调用。

```
f := func(x, y int) int { return x + y }
func(ch chan int) { ch <- ACK }(replyChan)
```

函数字面值是闭合的：他们可以访问定义在外围函数的变量。这些变量在外围函数和函数字面值之间共享，并且该变量生存期一直到它不再被访问。

## Primary expressions
主表达式是一元和二元表达式的操作数。

```
PrimaryExpr =
	Operand |
	Conversion |
	MethodExpr |
	PrimaryExpr Selector |
	PrimaryExpr Index |
	PrimaryExpr Slice |
	PrimaryExpr TypeAssertion |
	PrimaryExpr Arguments .

Selector       = "." identifier .
Index          = "[" Expression "]" .
Slice          = "[" [ Expression ] ":" [ Expression ] "]" |
                 "[" [ Expression ] ":" Expression ":" Expression "]" .
TypeAssertion  = "." "(" Type ")" .
Arguments      = "(" [ ( ExpressionList | Type [ "," ExpressionList ] ) [ "..." ] [ "," ] ] ")" .

```

```
x
2
(s + ".txt")
f(3.1415, true)
Point{1, 2}
m["foo"]
s[i : j + 1]
obj.color
f.p[i].x()
```

## Selectors
对于一个不是包名的主表达式x而言，选择器表达式为`x.f`

`x.f`表示值x的字段或者方法f。标识符f被称为选择器;选择器一定不能是下划线。选择器表达式的类型是f的类型。如果x是一个包的名字，则请参考[qualified ientifiers]()。

选择器f可以表示成一个T类型的字段或者方法f，它也可以引用T中嵌套的嵌入式字段[embedded field]()或者方法f。为了到达f而走过的嵌入式字段的数量被称作f在T中的深度。一个声明在T中的字段或者方法f的深度为0。A声明于T中，f声明于A中，则f在T中的深度为f在A中的深度加1。

以下规则适用于选择器：
 1. 对于一个T或者*T（T不是指针类型或者接口类型）类型的值x而言，x.f表示T的最浅深度的字段或者方法。如果T的最浅深度没有f，则该选择器是非法的。
 2. 对于一个I类型（I是一个接口类型）的值x而言，x.f表示x的动态值的方法f。如果I的方法集中没有方法f，则该选择器是非法的。
 3. 作为例外，如果x的类型是一个定义的指针类型并且(*x).f是一个代表某一字段的有效选择器表达式，则x.f是(*x).f的缩写。
 4. 如果x是一个指针类型，值为nil，并且x.f表示一个struct的字段，计算或者赋值给x.f会导致[runtime-time panic]()

例如，给予以下声明：

```
type T0 struct {
	x int
}

func (*T0) M0()

type T1 struct {
	y int
}

func (T1) M1()

type T2 struct {
	z int
	T1
	*T0
}

func (*T2) M2()

type Q *T2

var t T2     // with t.T0 != nil
var p *T2    // with p != nil and (*p).T0 != nil
var q Q = p
```

调用的时候可以使用如下写法：

```
t.z          // t.z
t.y          // t.T1.y
t.x          // (*t.T0).x

p.z          // (*p).z
p.y          // (*p).T1.y
p.x          // (*(*p).T0).x

q.x          // (*(*q).T0).x        (*q).x is a valid field selector

p.M0()       // ((*p).T0).M0()      M0 expects *T0 receiver
p.M1()       // ((*p).T1).M1()      M1 expects T1 receiver
p.M2()       // p.M2()              M2 expects *T2 receiver
t.M2()       // (&t).M2()           M2 expects *T2 receiver, see section on Calls
```
但是下面的写法是无效的：
```
q.M0()    // (*q).M0 is valid but not a field selector
```

##Method expressions
如果M是类型T的方法集之一，则T.M函数可以和M函数一样被正常调用，但是M函数会多出一个方法的接收者参数。

```
MethodExpr    = ReceiverType "." MethodName .
ReceiverType  = Type .
```
考虑一个有两个方法的结构体类型T，Mv是一个接收者参数为T类型的方法，Mp是一个接收者参数为*T类型的方法。如下：
```
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
```
表达式`T.Mv`会生成一个等价于Mv的函数，但是第一个显示参数为接收者;有如下形式`func(tv T, a int) int`

我们可以使用接收者参数来正常的调用该函数，因此，以下5个函数的调用是等价的：
```
t.Mv(7)
T.Mv(t, 7)
(T).Mv(t, 7)
f1 := T.Mv; f1(t, 7)
f2 := (T).Mv; f2(t, 7)
```
相似的，表达式`(*T).Mp`会生成如下的Mp函数`func(tp *T, f float32) float32`

对于一个带有接收者参数的方法而言，我们可以得到一个带有显示指针接收参数的函数，因此`(*T).Mv`会生成一个函数值，代表Mv的函数值表示如下:`func(tv *T, a int) int`

这样的函数会间接通过接收者参数来创建一个值，然后将这个值当作接收者参数来传递给底层的方法;该方法并不会重写这个把地址传递进来的变量值。

还有最后一种情况，将值接收者函数用于指针接收者方法是非法的。因为指针接收者方法并不存在于值类型的方法集中。

从方法中取得的函数值被称为函数调用语法;接收者被作为该函数调用的第一个参数。如，给予`f := t.Mv`，f可以以`f(t,7)`的形式调用，但是`t.f(7)`的调用是不正确的。如果想构成一个绑定接收者的函数，使用[function literal]()或者[method value]()。

从接口类型的方法中获取一个函数值是合法的。该函数值显示的使用该接口类型的接收者作参数。

##Method values
如果表达式x拥有静态类型T并且M是T类型的方法集，x.M被叫作方法值。方法值x.M是 一个可以被正常调用的函数值，并且拥有和方法x.M被调用时相同的参数。表达式x被计算和保存于方法值的计算期间。然后被保存的复制值会被用于任意调用函数的接收者参数，这些函数可以被稍后执行。

T可以是一个接口或者非接口类型。

正如上面[method expressions]中所说的一样，结构体类型T有两个方法，一个方法接收者类型是T，另一个方法的接收者类型是*T。

```
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
var pt *T
func makeT() T
```
表达式`t.Mv`会生成一个`func(int) int`类型的函数值，下面的两个调用是等价的：
```
t.Mv(7)
f := t.Mv; f(7)
```
相似的，表达式`pt.Mp`会生成一个`func(float32) float32`类型的函数值。

正如[selectors]()一样，使用指针对一个非接口类型的带有值接收者方法进行调用时，该指针将会自动被解引用：`pt.Mv`和`(*pt).Mv`是等价的。

正如[method calls]()一样，使用一个可寻址值来引用一个带有指针接收者的方法时，该可寻址值会被自动获取地址：`t.Mp`等价于(&t).Mp。

```
f := t.Mv; f(7)   // like t.Mv(7)
f := pt.Mp; f(7)  // like pt.Mp(7)
f := pt.Mv; f(7)  // like (*pt).Mv(7)
f := t.Mp; f(7)   // like (&t).Mp(7)
f := makeT().Mp   // invalid: result of makeT() is not addressable
```

尽管上面所举的例子全都是非接口类型的，但是从一个接口类型的值中创建一个方法值也同样合法。

```
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```

##Index expressions
一个最主要的表达式形式如下：
```
a[x]
```
它表示数组，指向数组的指针，slice，string或者map的一个元素，该元素由x索引。对应的，x的值被称作索引或者map key。适用于下面的规则：
如果a不是一个map:
 - 索引x必须是一个整形类型或者是一个没有类型的常量
 - 常量索引值必须是非负数并且可以被一个类型为int的值会表示
 - 一个没有类型的常量索引会被给予一个int类型
 - 索引x必须介于0到len(a)-1之间（包括边界），否则就出范围了

对于一个数组类型A的变量a而言:
 - 常量索引必须在a的长度范围内
 - 如果x在运行时超出范围，则会发生[run-time panic]()
 - a[x]是数组在x索引处的元素值，并且a[x]元素的类型是A的元素类型

对于一个指针数组类型的指针而言：
 - a[x]是(*a)[x]的缩写

对于一个slice类型S的值a而言：
 - 如果x在运行时超出了范围，则会发生[run-time panic]()
 - a[x]是在索引x处的slice元素，并且a[x]的类型是S的元素类型

对于一个string类型的a而言：
 - 如果a是一个常量，则常量索引必须在a的长度范围内
 - 如果x在运行时超出a的长度范围，则会发生[run-time panic]
 - a[x]是索引x处的非常量byte值，并且a[x]的类型是byte
 - a[x]不可以被赋值

对于一个map类型M的变量a而言：
 - x的类型必须可以赋值给M的key类型
 - 如果map包含了一个key为x的项，则a[x]是key为x的元素值，并且a[x]的类型为M的元素类型

除上之外，a[x]为非法操作。

应用于map[K]V类型的map上的索引表达式被用于赋值或者初始化，有以下特殊形式

```
v, ok = a[x]
v, ok := a[x]
var v, ok = a[x]
var v, ok T = a[x]
```

这些操作会生成一个另外的没有类型的boolean值。如果key为x的元素存在，则ok的值为true，否则为false。

向一个值为nil的map赋值会发生[run-time panic]()

##Slice expressions
slice表达式可以从一个string，array，指向array的指针或者slice中构建一个子串或者slice。有两种变种：一种是确定高低边界的简单形式，还有一种是以slice能力确定的边界的完全形式。

###Simple slice expressions
对于一个string，array，指向array的指针，或者slice a而言，主要的表达式为：
```
a[low:high]
```
该表达式会构建一个子串或者slice。low和high的索引值选择了a中哪些元素会出现在结果中。该结果的索引值从0开始，并且长度等于high-low。将array a切片后：

```
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]
```
slice s拥有[]int类型，长度为3，容量为4，元素分别如下：
```
s[0] == 2
s[1] == 3
s[2] == 4
```

为了方便起见，任意索引值都可以被省略。省略的low索引默认为0;省略的high索引默认为slice的长度：
```
a[2:]  // same as a[2 : len(a)]
a[:3]  // same as a[0 : 3]
a[:]   // same as a[0 : len(a)]
```
如果a是一个指向array的指针，a[low:high]为(*a)[low:high]的缩写。

对于array或者string而言，索引的范围是0<=low<=high<=len(a)，否则便超出范围。对于slice而言，上索引边界是slice的容量值cap(a)，而不是长度。常量索引必须是百负值并且可以被int类型的值来表示;对于array和常量string而言，常量索引值必须在范围内。如果索引值全是常量，则他们必须满足low<=high。如果运行时索引值超出范围，则会发生[run-time panic]()

如果一个有效的slice表达式的切片操作数是一个nil值的slice，则结果也是一个nil值的slice。否则，如果结果是一个slice，该结果共享原slice操作数的底层array。

###Full slice expressions
对于一个array，指向array的指针，或者slice a(不是string)而言，主要的表达式为`a[low : high : max]`

该表达式会构建一个相同类型的slice，并且和slice简单表达式a[low : high]拥有相同的长度和元素。另外，可以通过将容量设置为max-low来控制结果slice的容量。只有第一个索引值可以被省略;省略后默认为0。将array a切片后：
```
a := [5]int{1, 2, 3, 4, 5}
t := a[1:3:5]
```
slice t拥有[]int类型，长度为2，容量为4，元素分别是：
```
t[0] == 2
t[1] == 3
```
正如简单slice表达式一样，如果a是一个指向array的指针，a[low:high:max]为(*a)[low:high:max]的缩写。如果切片的操作数是一个array，则该array必须是可寻址的。

索引值在0<=low<=high<=max<=cap(a)的范围内，否则便会超出范围。常量索引值必须是非负数并且可以被一个int类型的值所表示;对于array而言，常量索引值必须在范围内。如果多个索引全是常量，则这些索引值必须相互都在范围内。如果索引值在运行时超出了范围，则会发生[run-time panic]

##Type assertions
对于一个接口类型的表达式x和类型T而言，类型断言的主要表达式如下：
```
x.(T)
```
该表达式断言x不是nil值并且存储在x中的值类型是T。符号x.(T)并称作类型断言。

更准确的说，如果T不是一个接口类型，x.(T)断言x的动态类型和类型T完全相同。在这种情况下，T必须实现了接口类型x;否则该类型断言无效，因为如果T没有实现接口类型x则x不可能存储T类型的值。如果T是一个接口类型，x.(T)断言x的动态类型实现了接口T。

如果类型断言成功的话，则表达式的值为存储在x中的实际值并且它的类型为T。如果类型断言失败，则会发生runtime panic。换句话说，即使x的动态类型只能在运行时知道，但是在一个正确的程序中，x.(T)的类型就是T。

```
var x interface{} = 7          // x has dynamic type int and value 7
i := x.(int)                   // i has type int and value 7

type I interface { m() }

func f(y I) {
	s := y.(string)        // illegal: string does not implement I (missing method m)
	r := y.(io.Reader)     // r has type io.Reader and the dynamic type of y must implement both I and io.Reader
	…
}
```
类型断言被使用于赋值或者特殊形式的初始化语句中。

```
v, ok = x.(T)
v, ok := x.(T)
var v, ok = x.(T)
var v, ok T1 = x.(T)
```

上面的类型断言会生成一个额外的没有类型的布尔值。如果断言成功则ok的值是true，如果断言失败则ok的值是false并且v的值是类型T对应的0值。这种情况下是没有panic发生的。

##Calls
给予一个函数类型F的表达式f，
```
f(a1, a2, ... an)
```
使用实参a1，a2，... an来调用函数f。除了特殊情况外，实参必须是单值表达式，可以赋值给F的形参类型并且在函数调用前被计算。该函数表达式的类型是F的结果类型。这和方法的调用是相似的，但是方法本身是一个选择器，在该选择器上有一个该方法接收者类型的值。

```
math.Atan2(x, y)  // function call
var pt *Point
pt.Scale(3.5)     // method call with receiver pt
```

在函数调用中，函数的值和实参以[in the usual order]()的顺序被计算。然后以值的形式传递给函数的形参，最后调用函数开始执行。当被调用函数返回的时候，该被调用 函数的返回参数也是以值的形式被传递到调用者函数。

调用一个值为nil的函数会发生panic

特殊情况，如果函数或者方法g和返回值数量和函数或者方法f的参数数量相同，并且可以相互之间是可以赋值的，则调用f(g(parameters_of_g))将会在调用g后，将g的返回值按顺序绑定到f的形参上。此时，f的函数调用除了包括g函数以外不能包括任何参数，并且g函数必须至少有一个返回值。如果f最后有一个...参数，则在g的返回值被赋值给f的形参后，剩下的返回值会被赋给...形参。

```
func Split(s string, pos int) (string, string) {
	return s[0:pos], s[pos:]
}

func Join(s, t string) string {
	return s + t
}

if Join(Split(value, len(value)/2)) != value {
	log.Panic("test fails")
}
```
如果x的方法集包括m并且参数列表可以被赋值给m的形参列表，则方法调用x.m()便是有效的。如果x是可寻址的并且&x的方法集包括m，则x.m()便是(&x).m()的缩写。

```
var p Point
p.Scale(3.5)
```

不存在明确的方法类型，也不存在方法字面值。

##Passing arguments to ... parameters
如果f是一个变参函数，f的最后一个形参p的类型是...T，则在f内，p的类型等同于[]T。如果调用f的时候没有p对应的实参，则被传递给p的值为nil。否则，被传递给p的值是一个新的类型为[]T的slice，该slice带有一个新的底层数组，该底层数组的连续元素为实际的参数，这些参数必须可以被赋值给T。该slice的长度和容量便是被限制的p的参数数量，并且可能在每次调用中都是不同的。

给予以下函数调用：
```
func Greeting(prefix string, who ...string)
Greeting("nobody")
Greeting("hello:", "Joe", "Anna", "Eileen")
```
在函数Greeting内，第一次调用时，who的值为nil，第二次调用中值为[]string{"Joe", "Anna", "Eileen"}。

如果最后的参数可赋值给一个类型为[]T的slice，则该参数可以不经改变的被赋值给...T形参，如果实参后面跟了...，在这种情况下没有新的slice被创建。

给予以下slice s和函数调用 ：
```
s := []string{"James", "Jasmine"}
Greeting("goodbye:", s...)
```
在Greeting函数内，将会拥有和s的底层数组一样的slice值

##Operators
操作符将操作数合并成了表达式。

```
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .

binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .

unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .****
```

比较操作在另外的地方讨论。对于其他的二元操作符，操作数的类型必须完全相同，除非操作涉及到了移位或者无类型的常量。对于只涉及常量的操作，请看[constant expressions]()节。

除了移位操作之外，如果一个操作数是无类型的并且剩下的操作数不是无类型的，则该无类型常量将会被转换成其它操作数的类型。

在移位表达式右操作数必须是无符号的整数类型，或者是一个可以被整形数值表示的无符号的常量值。如果一个非常量移位表达式的左操作数是一个没有类型的常量，并且如果该移位表达式可以它的被左操作数单独的替换，则该左操作数会被首先转换成默认的类型。

```
var s uint = 33
var i = 1<<s                  // 1 has type int
var j int32 = 1<<s            // 1 has type int32; j == 0
var k = uint64(1<<s)          // 1 has type uint64; k == 1<<33
var m int = 1.0<<s            // 1.0 has type int; m == 0 if ints are 32bits in size
var n = 1.0<<s == j           // 1.0 has type int32; n == true
var o = 1<<s == 2<<s          // 1 and 2 have type int; o == true if ints are 32bits in size
var p = 1<<s == 1<<33         // illegal if ints are 32bits in size: 1 has type int, but 1<<33 overflows int
var u = 1.0<<s                // illegal: 1.0 has type float64, cannot shift
var u1 = 1.0<<s != 0          // illegal: 1.0 has type float64, cannot shift
var u2 = 1<<s != 1.0          // illegal: 1 has type float64, cannot shift
var v float32 = 1<<s          // illegal: 1 has type float32, cannot shift
var w int64 = 1.0<<33         // 1.0<<33 is a constant shift expression
var x = a[1.0<<s]             // 1.0 has type int; x == a[0] if ints are 32bits in size
var a = make([]byte, 1.0<<s)  // 1.0 has type int; len(a) == 0 if ints are 32bits in size
```

### Operator precedence
一元运算符拥有最高的优先级。`++`和`--`操作符形式的语句不是表达式，他们不在操作符的层次之内。语句*p++和(*p)++是一样的。

二元操作符有5种优先级。乘法运算符的优先级最高，紧接着是加法运算符，比较运算符，&&逻辑与运算符和最后的||逻辑或运算符。

相同优先级的二元运算符从左到右依次计算。例如，x/y*z和(x/y)*z是一样的。

```
+x
23 + 3*x[i]
x <= f()
^a >> b
f() || g()
x == y+1 && <-chanPtr > 0
```

## Arithmetic operators
算术运算符应用于数字值运算，会生成和第一个操作数相同类型的结果。四个标准的算术运算符+，-，*，/应用于整数，符点数和复数运算;+运算也可以应用于字符串运算。按位的逻辑移位和算数移位只适用于整数。

```
+    sum                    integers, floats, complex values, strings
-    difference             integers, floats, complex values
*    product                integers, floats, complex values
/    quotient               integers, floats, complex values
%    remainder              integers

&    bitwise AND            integers
|    bitwise OR             integers
^    bitwise XOR            integers
&^   bit clear (AND NOT)    integers

<<   left shift             integer << unsigned integer
>>   right shift            integer >> unsigned integer
```

###Integer operators
对于两个整数值x和y而言，整数的商q = x / y，余数r = x % y，他们满足下面的关系：
```
x = q*y + r and |r| < |y|
```
x / y 结果会向0方向舍入["truncated division"](https://en.wikipedia.org/wiki/Modulo_operation)
```
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
```
该规则的例外情况是，如果被除数x的大小是int所代表类型的负最小值，商值q = x / -1 等于x并且余数r = 0，这是由二进制补码整型溢出导致的。
```
			 x, q
int8                     -128
int16                  -32768
int32             -2147483648
int64    -9223372036854775808
```

如果除数是一个常量，则它必须非0。如果运行时除数为0,则会发生panic。如果被除数是非负值并且除数是一个2的平方常量，则该除法运算可以由移位运算所替代，计算余数可以由位与运算替代。
```
 x     x / 4     x % 4     x >> 2     x & 3
 11      2         3         2          3
-11     -2        -3        -3          1
```
移位操作通过移位右操作数所指定的个数来移位左操作数。如果左操作数是一个有符号的，则为算术移位。如果左操作数无符号数，则为逻辑移位。对于移位的个数来说，没有什么最高的上限。移动n位就行人于移动1位移动了n次。结果就是x << 1 和 x*2是相同的;并且x >> 1 和x/2是相同的，但是会向负无穷方向舍入。

对于整型操作数而言，一元操作符+,-和^定义如下：
```
+x                          is 0 + x
-x    negation              is 0 - x
^x    bitwise complement    is m ^ x  with m = "all bits set to 1" for unsigned x
                                      and  m = -1 for signed x
```

### Interger overflow
对于无符号整型值而言，操作符+，-，*和<<都是以2的n次方为模来计算的，其中n是无符号整型值的位宽。不严格的说，一旦溢出发生，则这些无符号整型值操作会忽略高比特位，并且这个时候程序会依赖于"wrap around"

对于有符号的整型数而言，操作符+,-,*,/和<<也会发现溢出并且确切的结果会由该有符号整型数的表示范围，操作符和操作数所决定。这时候的溢出不会发现异常。编译器不能在假设溢出不会发现的情况下来优化代码。例如，编译不可以认为x < x + 1总是正确的。

### Floating-point operators
对于浮点数和复数而言，+x和x是一样的，而-x是x的相反值。根据IEEE-754标准，浮点数或者复数被0除的结果是不确定的;这时候是否会发现panic是和具体实现相关的。

在具体的实现中，多个浮点数操作可能会被合并成单个融合后的操作，合并操作有可能会跨语句发生，并且个别情况下会生成一个不同于执行和舍入指令所产生的值的值。浮点类型的转换会明确的舍入到目标类型的精度，阻止操作融合会忽略舍入操作。

例如，一些架构会提供一个FMA(融合乘和加)指令，对于x*y+z的计算中，该FMA指令会在不舍入中间结果x*y的情况下计算。下面的例子展示了在什么情况下go程序会使用该指令:

```
// FMA allowed for computing r, because x*y is not explicitly rounded:
r  = x*y + z
r  = z;   r += x*y
t  = x*y; r = t + z
*p = x*y; r = *p + z
r  = x*y + float64(z)

// FMA disallowed for computing r, because it would omit rounding of x*y:
r  = float64(x*y) + z
r  = z; r += float64(x*y)
t  = float64(x*y); r = t + z
```

###String concatenation
可以使用+操作符或者+=赋值操作符来连接strings:
```
s := "hi" + string(c)
s += " and good bye"
```
string类型的加操作通过连接各操作数来形成一个新的string

