# Types
类型确定了一系列的值和针对这些值的操作和方法。类型可以类型名来表示，类型名可以使用type关键字来表示，该关键字会从已有类型中产生一个新的类型。

```
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
```

已经被命名的布尔，数字和字符串类型是预声明的。名字的命名类型由type[type declarations]()关键字来指定。组合类型--数组，结构体，指针，函数，接口，slice，map和channel类型--可以使用type字面值被重新构建。

每一个类型T拥有一个底层类型:如果T是预声明类型布尔，数字，字符串，或者是类型字面值其中之一，则相对的底层类型就是T本身。否则，T的底层类型就是T在[type declartion]()中引用到的类型类型的底层类型。

```
type (
	A1 = string
    A2 = A1
)

type (
	B1 string
    B2 B1
    B3 []B1
    B4 B3
)
```

string，A1，A2，B1和B2的底层类型是string。[]B1，B3和B4的底层类型是[]B1

## Method sets
类型可以拥有方法集和它绑定到一起。接口类型的方法集就是它的接口。其它T类型的方法集由所有接收类型为T的方法组成。对应指针类型*T的方法集为所有接收类型为*T或者T的方法（这代表，*T的方法集包括T的方法集）。更深层次的规则也应用于包含内嵌字段的结构体，正如[struct types]()一节中描述的一样。任何其它的类型拥有空的方法集。在一个方法集中，每一个方法必须有一个独一无二的非空方法名字。

类型的方法集决定了类型实现的接口，并且决定了用该类型作接收值可以调用的方法。

## Numeric types
数字类型代表了一系列的整型或者浮点类型值。预声明的并且与架构无关的数字类型为：

```
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32
```

一个n个比特位的整型值是n比特位宽的并且使用[tow's complement arithmetic]()来表示。

同样也有一些预声明的数字类型，他们有确切的长度。

```
uint	either 32 or 64 bits
int 	same size as uint
uintptr	an unsigned integer large ennough to store the uninterpreted bits of a pointer value
```

为了避免可移植性问题，除byte类型外，所有的数字类型全是不同的。byte是uint8的别名，rune是int32的别名。当不同的数字类型混合在了同一个表达式或者赋值语句中的时候，需要相互转换。例如，即使在同一架构下int32和int拥有相同的大小，但是它们也是不同的类型。

## String types
string类型代表了string值的集合。string值是（有可能为空）了系列的字节序列。string的值是不可更改的：一旦创建，便不可能去更改它的内容。预声明的string类型是string

我们可以使用内建函数len来获取一个string s的长度(它的字节数)。如果该string s是一个常量，则该string s的长度是一个编译时常量。我们可以使用整形的[indices]从0到len(s)-1来获取一个string的各字节。如果s[i]是string的第i个字节，那么&s[i]是无效的。

## Array types
数组是一个被编号的单一类型的元素序列，该单一类型被称作元素类型。元素的数量被称作数组的长度，并且永远不会是负数。

```
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

长度也属于数组类型的一部分;该长度值必须是一个非负常量，并且可以被一个int类型的值所表示。数组的长度可以使用内建函数len来获取。所有的数组元素可以使用整形[indices]()0到len(a)-1来一一获取。数组的类型是总是一维的，但是可以被组合形成多维类型。

```
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

## Slice types
slice是一个描述符，该描述符表示一个底层数组的连续的段并且提供了方法来访问该底层数组。slice类型表示了所有由数组元素类型组成的slice。未初始化的slice值为nil。

`SliceType = "[" "]" ElementType .`

就像数组一样，slice是可以索引的并且拥有长度。slice s的长度可以使用内建函数len来获取;但是不同于数组的是，该slice的长度在运行期间是可以改变的。所有的slice元素可以使用整形[indices]()0到len(s)-1来获取。一个slice元素的索引可以小于相同的元素在底层数组中的索引。

slice底层的数组可以延伸到slice的末尾。容量值即为衡量该延伸的计量值：该容量是slice长度和超过slice长度之外的底层数组长度之和;一个长度到达该容量的slice可以通过从原先的slice[slicing]()一个新的slice来被创建。slice的容量可以使用内建函数cap来获取。

我们使用内建函数[make]()来创建一个新的，被初始化的，元素类型为T的slice，该make函数接收三个参数，第一个是slice类型，第二个是slice长度，第三个是可选的slice的容量 。一个被make创建的slice总是会被分配一个新的隐藏的底层数组，并且返回引用该底层数组的slice值。

make([]T, length, capacity) `

下面两个方法的效果是相同的：
```
make([]int, 50, 100)
new([100]int)[0:50]
```

正如数组一样，slice总是一维的，但是可以以数组的数组的形式被组合成更高维度的对象。在数组的数组这种形式中，内部的数组总是拥有相同的长度;在slice的slice（或者数组的slice）这种形式中，内部的slice的长度可以动态改变的。此外，slice的slice(或者数组的slice)中，内部的slice必须被逐一单独初始化。

## Struct types
struct是命名元素的集成，这些元素被叫作fields，每一个field都拥有一个名字和类型。field名字可以被显示指定或者隐式指定。在struct内部，非空field的名字必须是独一无二的。

```
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .
```

```
// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}
```

一个被声明类型但是没有名字的field被称作内嵌式field。内嵌式的field必须被由该类型名字T或者一个指向非接口类型的类型名字*T来指定，并且T本身不能是一个指针类型。该名字作为field名字

```
// A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
```

由于field名字的独一无二性，下面的声明是非法的。

```
struct {
	T     // conflicts with embedded field *T and *P.T
	*T    // conflicts with embedded field T and *P.T
	*P.T  // conflicts with embedded field T and *T
}
```

如果在一个struct中，表示该field或者方法的x.f是合法的[selector]()，则该内嵌式的field或者方法被称作promoted

promoted fields 像变通的struct field一样，但是，在struct的[composite literals]()中他们不能被用作field名字。

```
type T int
type abc struct {
	[]*T     这样使用是非法的
}
```

给予一个struct类型S和一个类型名T，promoted方法如下被包括在struct的方法集中：
 - 如果s包含内嵌式field T，S和*S的方法集都包括接收者为T的promoted方法。*S的方法集同样也包括接收者为*T的promoted方法
 - 如果S包含一个内嵌式field *T，S和*S的方法集都包括接收者为T或者*T的promoted 方法。

一个field声明可以紧跟着一个可选的字符串字面值标签，该标签会变成对应字段声明中字段的属性。空标签字符串等同于没有标签。该标签可以被[reflection interface]()来获取，并且在[type identity]中扮演重要的角色，但是其它方面它是被忽略的。

```
struct {
	x, y float64 ""  // an empty tag string is like an absent tag
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// A struct corresponding to a TimeStamp protocol buffer.
// The tag strings define the protocol buffer field numbers;
// they follow the convention outlined by the reflect package.
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```

## Pointer types
指针类型表示所有指向某类型变量的集合，某类型被称作该指针的基础类型。未初始化的指针为nil。

```
PointerType = "*" BaseType .
BaseType    = Type .
```

```
*Point
*[4]int
```

## Function types
函数类型表示拥有相同的参数和结果类型的集合。一个未初始化的函数类型变量值为nil。

```
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

在函数的参数或者结果列表中，参数或者结果的名字必须或者全存在，或者全部不存在。如果存在，每一个名字代表一项确定类型的参数或者结果，并且所有的非空名字必须是独一无二的。如果不存在参数或者结果名字，每一个类型代表该类型的一个条目。参数和结果列表总是由括号括起来的，除非只有一个未命名的结果，则它可以被写作不用括号括起来的类型。

函数的最后一个参数可以拥有一个...类型前缀，一个带有这种参数的函数被称作*variadic*(变参函数)，使用该函数的时候，对应的...参数可以传递0个或者多个参数。

```
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

## Interface types
确定了方法集的接口类型被称作它的接口。一个接口类型的变量可以存储任何值，只要该值类型的方法值是该接口的子集。这样的类型实现了该接口。一个未初始化的接口类型的变量值为nil。

```
InterfaceType      = "interface" "{" { MethodSpec ";" } "}" .
MethodSpec         = MethodName Signature | InterfaceTypeName .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

在一个接口类型中的所有方法集中，每一个方法必须拥有独一无二的名字。

```
// A simple File interface
interface {
	Read(b Buffer) bool
	Write(b Buffer) bool
	Close()
}
```

不只一种类型可以实现相同的一个接口。例如，如果两个类型S1和S2有如下方法集

```
func (p T) Read(b Buffer) bool { return … }
func (p T) Write(b Buffer) bool { return … }
func (p T) Close() { … }
```

这上面的方法集中，T表示S1或者S2，这就表示File接口被S1和S2实现了，而不管S1和S2是否还有其它的方法集。

一个类型实现了任意包含该类型方法子集的接口，因此可以实现多个不同的接口。例如，所有的类型实现了空接口：

`interface{} `

同样的，考虑下面的接口说明，该声明出现在一个[type delaration]内，被定义为一个叫Locker的接口。

```
type Locker interface {
	Lock()
    Unlock()
}
```
如果S1和S2同样实现了下面的方法
```
func (p T) Lock() { … }
func (p T) Unlock() { … }
```
则，S1和S2不仅实现了Locker接口，同样实现了File接口

接口T可以使用一个接口名字为E的接口来代替方法说明。这种形式被叫作T中的内嵌式接口E;它将把E的所有方法（导出的，非导出的）添加到接口T中。

```
type ReadWriter interface {
	Read(b Buffer) bool
	Write(b Buffer) bool
}

type File interface {
	ReadWriter  // same as adding the methods of ReadWriter
	Locker      // same as adding the methods of Locker
	Close()
}

type LockedFile interface {
	Locker
	File        // illegal: Lock, Unlock not unique
	Lock()      // illegal: Lock not unique
}
```

接口类型T不可以内嵌它本身，或者任何内嵌了T的接口，防止递归内嵌。

```
// illegal: Bad cannot embed itself
type Bad interface {
	Bad
}

// illegal: Bad1 cannot embed itself using Bad2
type Bad1 interface {
	Bad2
}
type Bad2 interface {
	Bad1
}
```

## Map types
map是一个未排序的同种类型的元素组，该类型就是元素的类型。map由一系列的唯一的其它类型的key来索引，这些类型被叫作key类型。未初始化的map变量值为nil。

```
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

key类型的操作数必须定义实现了[comparison operations]()中的== 和!=;因此，key类型肯定不是function, map或者slice。如果key是一个接口类型，这些接口类型的比较操作必须被提前定义;失败将会造成[run-time panic]()

```
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

map元素的数量为map的长度。对于一个map m来讲，我们可以使用内建函数len来获取m的长度，当然m的长度在运行期间是可变的。在运行期间，可以使用赋值[assignments]()来添加元素，使用[index expressions]来获取元素;我们可以使用内建函数delete来删除这些元素。

我们使用内建函数[make]()来创建一个新的，空的map，该make函数接收两个参数，第一个参数是map的类型，第二个参数是可选的容量参数

```
make(map[string]int)
make(map[string]int, 100)
```

初始化的map容量并不会超过map的大小：map会自己增长来容纳存储在其中的元素，值为nil的map是一个例外。一个值为nil的map就等同于一个空的map，但是值为nil的map是不能添加元素的。

## Channel types
channel通过send和receive一个指定类型的值的操作为[concurrently executing functions]()(并发执行函数)的通信提供了机制。未初始化的channel变量为nil。

`ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
`

可选的<- 操作符指定了channel的方向，send还是receive。如果没有指定方向，则该channel的通信方式是双向的。channel通过[conversion]()或者[assignment]()可以被限制为只send或者只receive。

```
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
```

<- 操作符将最左边的channel连接起来：

```
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
```

我们可以使用内建函数[make]()来初始化一个新的channel变量，该make函数接收两个参数，第一个参数是channel的类型，第二个参数是可选的容量参数：

`make(chan int, 100)`

容量代表元素的数量，设置该值会设置下channel的buffer。如果该容量参数为0或者不存在，则该channel是不带buffer的，并且只有当发送都或者接收者准备好的时候，通信才会成功。否则，该channel就是带buffer的，并且只有当该channel非满(对于发送者而言)或者非空(对于接收者而言)并且非堵塞的时候，通信才会成功。对一个值为nil的channel执行通信操作将会永远堵塞。

[close]()函数可以关闭channel。[receive operator]接收操作的多值赋值可以检测该channel是否被关闭。

单个channel是多个goroutines安全的。在多个goroutines中，我们可以随意使用[send statements]，[receive operations]，cap和len内建函数。channel的行为就好像先进先出队列。例如，如果一个goroutine向一个channel发送了值并且第二个goroutine接收了这些值，这些接收的值的顺序和发送值的顺序是相同的。


