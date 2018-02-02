# Built-in functions
内建函数是预先声明的。他们就像正常的函数调用一样，但是其中一部分接收一个类型值而不是表达式作为第一个参数。

内建函数没有go标准类型，因此他们只能出现在[call expressions]();不能被当作函数值来使用。

## Close
对于一个channel c而言，内建函数close(c)表示不再有值可以被写入该channel。如果c是一个只读的channel的话，该close(c)操作便是一个错误操作。向一个已经关闭的channel执行发送或者close操作会造成[run-time panic]()。close nil channel也会造成[run-time panic]()。在调用了close后，并且所以之前发送的值也已经被收到后，接收操作将会非堵塞的返回channel类型的0值。多值接收操作会返回接收到的值，并且也会返回通道是否关闭的标识。

## Length and capacity
内建函数len和cap接受各种类型的参数，并且返回一个int类型的结果。该函数的实现保证了函数的回返结果总是int类型。

```
Call      Argument type    Result

len(s)    string type      string length in bytes
          [n]T, *[n]T      array length (== n)
          []T              slice length
          map[K]T          map length (number of defined keys)
          chan T           number of elements queued in channel buffer

cap(s)    [n]T, *[n]T      array length (== n)
          []T              slice capacity
          chan T           channel buffer capacity
```

slice的容量是指所有的元素的数量，这些元素所占空间分配在了底层的数组中。在任意时刻存在下面的关系。

`0 <= len(s) <= cap(s)`

如果一个slice, map 或者channel的值是nil，则他们的长度都是0。如果一个slice和channel的值为0,则他们的容量为0。(cap不支持map操作)

如果s是一个字符串常量，则表达式len(s)代表一个常量。如果s的类型是一个数组或者是一个指向数组的指针并且表达式s不包含channel接收操作或者函数调用，刚len(s)和cap(s)表达式都是常量;在这种情况下，s是不需要被计算的。除以上这两种情况外，len和cap的函数调用不是常量并且s也会被计算。

```
const (
	c1 = imag(2i)                    // imag(2i) = 2.0 is a constant
	c2 = len([10]float64{2})         // [10]float64{2} contains no function calls
	c3 = len([10]float64{c1})        // [10]float64{c1} contains no function calls
	c4 = len([10]float64{imag(2i)})  // imag(2i) is a constant and no function call is issued
	c5 = len([10]float64{imag(z)})   // invalid: imag(z) is a (non-constant) function call
)
var z complex128
```

## Allocation
内建函数new接收一个类型T作为参数，然后运行的时候会为这个类型的变量分配空间，并且返回*T的类型值指向分配的空间。变量的初始化如[initial values]()中的描述一样。

`new(T)`

例如：

```
type S struct {a int; b float64}
new(S)
```

以上操作为类型为S的变量分配内存，然后初始化该空间（a=0, b=0.0），并返回包含该分配空间地址的*S类型的值。

## Making slices, maps and channels

内建函数make接收一个类型T作为第一个参数，该类型必须是slice,map或者channel，后面可选择性的加上与类型相关的参数列表。该函数会返回T类型的值（非*T类型）。分配的内存会被初始化为[initial values]()中描述的内容。

```
Call             Type T     Result

make(T, n)       slice      slice of type T with length n and capacity n
make(T, n, m)    slice      slice of type T with length n and capacity m

make(T)          map        map of type T
make(T, n)       map        map of type T with initial space for approximately n elements

make(T)          channel    unbuffered channel of type T
make(T, n)       channel    buffered channel of type T, buffer size n
```

size参数n和m必须是整型类型或者非类型化的。如果size参数是一个常量，则必须是非负数而且也可以被表示为int类型。如果n和m都被提供了并且全是常量，则n必须小于等于m。如果运行时n是一个负数或者比m大，刚[run-time panic]()会出现。

```
s := make([]int, 10, 100)       // slice with len(s) == 10, cap(s) == 100
s := make([]int, 1e3)           // slice with len(s) == cap(s) == 1000
s := make([]int, 1<<63)         // illegal: len(s) is not representable by a value of type int
s := make([]int, 10, 0)         // illegal: len(s) > cap(s)
c := make(chan int, 10)         // channel with a buffer size of 10
m := make(map[string]int, 100)  // map with initial space for approximately 100 elements
```

以参数map类型和size大小为n调用make函数会创建一个初始化好空间并且可以保存n个map元素的map。

更加确切的行为是依赖于具体的实现的。

## Appending to and copying slice
内建函数append和copy协助完成基本的slice操作。结果不受参数引用的内存是否相交的影响。

[variadic]()函数append向类型为S的s变量中追加一个或多个值x，类型S必须是slice，并且会返回slice类型的结果值，也就是类型S。变量x被传递给类型为...T的参数，T是S的元素类型并且遵循[parameter passing rules]()规则。存在特殊的例子，append也会接收第一个参数是[]byte，第二个参数为string类型并且后面跟着...，这种形式的意思是将string追加到[]byte中

`append(s S, x ...T) S // T是S的元素类型`

如果s的容量不足以容纳追加后的所有值，append函数会申请一个新的，足够大的底层数组来满足s已经存在的slice元素和即将追加的值。否则，append会重复使用原先底层的数组

```
s0 := []int{0, 0}
s1 := append(s0, 2)                // append a single element     s1 == []int{0, 0, 2}
s2 := append(s1, 3, 5, 7)          // append multiple elements    s2 == []int{0, 0, 2, 3, 5, 7}
s3 := append(s2, s0...)            // append a slice              s3 == []int{0, 0, 2, 3, 5, 7, 0, 0}
s4 := append(s3[3:6], s3[2:]...)   // append overlapping slice    s4 == []int{3, 5, 7, 2, 3, 5, 7, 0, 0}

var t []interface{}
t = append(t, 42, 3.1415, "foo")   //                             t == []interface{}{42, 3.1415, "foo"}

var b []byte
b = append(b, "bar"...)            // append string contents      b == []byte{'b', 'a', 'r' }
```

函数copy会将slice元素从源地址src复制到目的地址dest并且返回被复制的元素的数量。该函数的两个参数类型必须一致并且可以赋值给一个[]T类型的slice。被复制的元素的数量从len(src)和len(dst)中取最小值。存在特殊例子，copy函数可以接收目的参数为[]byte类型，源参数为string类型。这种形式的意思是将string内的所有内容复制到byte类型的slice中。

```
copy(dst, src []T) int
copy(dst []byte, src string) int
```

例如：
```
var a = [...]int{0, 1, 2, 3, 4, 5, 6, 7}
var s = make([]int, 6)
var b = make([]byte, 5)
n1 := copy(s, a[0:])            // n1 == 6, s == []int{0, 1, 2, 3, 4, 5}
n2 := copy(s, s[2:])            // n2 == 4, s == []int{2, 3, 4, 5, 4, 5}
n3 := copy(b, "Hello, World!")  // n3 == 5, b == []byte("Hello")
```

## Deletion of map elements

内建函数delete会根据key将元素从map中移除。key的类型必须和map的key类型保持一致。

`delete(m, k) // 从map m中删除元素m[k]`

如果map m为nil或者元素m[k]不存在，delete不作任何操作。

## Manipulating complex numbers

有三个函数来组成分解复数。内建函数complex可以将一个浮点类型的实部和虚部组成一个复数值，而real和imag函数可以从一个复数值中解析出来实部和虚部

```
complex(realPart, imaginaryPart floatT) complexT
real(complexT) floatT
imag(complexT) floatT
```

参数的类型和返回值相对应。对于complex函数而言，该函数的两个参数必须所有相同的浮点类型而且返回值为所有对应浮点数参数的复数;如果返回值是complex64则对应的浮点类型为float32，complex128对应的是float64。如果参数的其中之一计算得到的结果是非类型化常量，它首先会被转化成别一个参数的类型。如果两个参数都是非类型化的常量，他们必须是非复数值或者虚部部分必须是0,并且函数的返回值会是一个非类型化的复数常量。

对于real和imag函数来说，参数必须是复数类型，并且回返类型是对应的浮点类型：float32对应complex64类型参数，float64对应complex128类型参数。如果参数是一个非类型化的常量，它必须是一个数字，并且函数的返回值是一个非类型化的浮点数常量。

real和imag函数一直构成了complex函数的反转，因此对于一个complex类型Z的值z而言，z == Z(complex(real(z), imag(z)))。

如果这些函数的操作数全是常量，返回值也一定是常量

```
var a = complex(2, -2)             // complex128
const b = complex(1.0, -1.4)       // untyped complex constant 1 - 1.4i
x := float32(math.Cos(math.Pi/2))  // float32
var c64 = complex(5, -x)           // complex64
var s uint = complex(1, 0)         // untyped complex constant 1 + 0i can be converted to uint
_ = complex(1, 2<<s)               // illegal: 2 assumes floating-point type, cannot shift
var rl = real(c64)                 // float32
var im = imag(a)                   // float64
const c = imag(b)                  // untyped constant -1.4
_ = imag(3 << s)                   // illegal: 3 assumes complex type, cannot shift
```

## Handling panics

两个内建函数panic和recover，协助报告控制[run-time panics]()和程序定义的错误条件。

```
func panic(interface{})
func recover() interface{}
```

当执行函数F的时候，对panic函数显示的调用或者一个[run-time panic]会终止函数F的执行。然后任何被F defer的函数会被正常执行。接下来，任何被调用F的函数defer的函数会被执行，这么一直下去直到该执行中的goroutine内所有被上层函数defer的函数全部执行完。在panic的那一点，程序被终止并且错误条件包括传递给panic的参数会被上报。这样的终止序列被叫做panicking

```
panic(42)
panic("unreachable")
panic(Error("cannot parse"))
```

recover函数允许程序控制一个panicking程序的行为。假设函数G defer了一个函数D，D函数调用了recover函数，此时一个在G正在执行的gorougine中的函数panic了。当被运行的被defer的函数到了D的时候，D函数调用recover函数的返回值将会是panic时传递给panic的参数值。如果在没有panic的情况下D正常返回，panicking序列会停止。在panic的情况下，在G函数和调用panic函数之间的函数状态将会被忽略，正常的执行会重新开始。任何在D之前被G defer的函数然后会被运行，G通过返回到调用它的函数而终止运行。

如果任何下面的条件发生，则recover函数的返回值为nil
- panic函数的参数为nil
- goroutine没有发生panicking
- recover函数没有直接被一个defer的函数调用

下面例子中的protect函数调用了函数g并且保护protect的函数调用者受到g函数引起的panic影响

## Bootstrapping

当前golang实现中在引导过程中提供了多个内建函数。为了完善性而将这些函数记录在文档中，但是并不能保证会他们会一直存在于golang语言中。他们不返回结果

```
Function   Behavior

print      prints all arguments; formatting of arguments is implementation-specific
println    like print but prints spaces between arguments and a newline at the end
```

实现约束：print和println函数不需要接收任意类型的函数，但是必须支持能够输出布尔，数字和字符串类型。