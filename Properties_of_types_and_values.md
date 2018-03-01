# Properties of types and values
## Type identity

两个类型要么是完全相同的，要么是不同的。

一个[defined type]()总是不同于其他类型的。另外，如果两个类型的底层类型字面值相构造相同，则这两个类型是完全相同的;也就是说，他们拥有相同的结构和对应的成分，拥有完全相同的类型。详细如下：
 - 如果两个数组拥有相同的元素类型和相同的数组长度，则这两个数组的类型是完全相同的。
 - 如果两个slice拥有完全相同的元素类型，则这两个slice的类型是完全相同的
 - 如果两个结构体拥有相同的字段序列，并且每个字段对应的名字，类型，标签完全相同，则这两个结构体的类型是完全相同的。不同包中的各结构体的非导出字段总是不相同的。
 - 如果两个指针拥有相同的基础类型，则这两个指针的类型是完全相同的。
 - 如果两个函数拥有相同数量的参数和返回值结果，对应的参数和返回值结果的类型完全相同，并且这两个函数都是变长参数或者都不是变长参数，则这两个函数的类型是相同的。参数的返回值结果的名字不作匹配。
 - 如果两个接口拥有相同的方法集，并且所有的方法拥有相同的名字和完全相同的函数类型，则这两个接口的类型是完全相同的。不同包中的接口的非导出方法总是不相同的。忽略方法集的顺序。
 - 如果两个map的key和元素的类型完全相同，则这两个map的类型完全相同。
 - 如果两个channel的元素类型和方向完全相同，则这两个channel的类型完全相同。

给予以下声明

```
type (
	A0 = []string
	A1 = A0
	A2 = struct{ a, b int }
	A3 = int
	A4 = func(A3, float64) *A0
	A5 = func(x int, _ float64) *[]string
)

type (
	B0 A0
	B1 []string
	B2 struct{ a, b int }
	B3 struct{ a, c int }
	B4 func(int, float64) *B0
	B5 func(x int, y float64) *A1
)

type	C0 = B0
```

由上可知，以下类型是完全相同的：

```
A0 A1 []string
A2 struct{a, b int}
A3 int
A4 A5 func(int, float64) *[]string

B0 and C0
[]int and []int
struct{ a, b *T5 } and struct{ a, b *T5 }
func(x int, y float64) *[]string, func(int, float64) (result *[]string), and A5
```

B0和B1是不同的，因为这两个类型是由不同的[type definitions]()创建的新类型;func(int, float64) *B0和func(x int, y float64) *[]string是不同的，因为B0不同于[]string

## Assignability
如果满足以下条件，则x可以被赋值给一个T类型的变量：
 - x的类型完全相同于T。
 - x的类型V和T拥有完全相同的底层类型[underlying types]()并且V和T至少有一个不是定义的新类型。
 - T是一个接口类型，并且x实现了T。
 - x是一个双向channel，T是channel类型的，x的类型V和T拥有完全相同的元素类型，并且V和T至少有一个不是定义的新类型。
 - x是预声明的标识符nil并且T是一个指针，函数，slice，map，channel，或者接口类型
 - x是一个未类型化的可被类型T表示的常量值[constant representable]。

## Representability
如果满足以下条件，则常量x可以被类型T所表示：
 - x包含于T类型所决定的值中[determined by T]()
 - T是浮点类型并且在没有溢出的情况下x可以四舍五入为T类型的精度。四舍五入使用IEEE 754规则，但是IEEE规定的负0被简化为一个无符号0值。注意常量值永远不会产生一个IEEE规定的负0，NaN值，或者无穷大值。
 - T是一个复数，并且x的组成中实部和虚部可以被T的对应组成类型表示(float32或者float64)

```
x                   T           x is representable by a value of T because

'a'                 byte        97 is in the set of byte values
97                  rune        rune is an alias for int32, and 97 is in the set of 32-bit integers
"foo"               string      "foo" is in the set of string values
1024                int16       1024 is in the set of 16-bit integers
42.0                byte        42 is in the set of unsigned 8-bit integers
1e10                uint64      10000000000 is in the set of unsigned 64-bit integers
2.718281828459045   float32     2.718281828459045 rounds to 2.7182817 which is in the set of float32 values
-1e-1000            float64     -1e-1000 rounds to IEEE -0.0 which is further simplified to 0.0
0i                  int         0 is an integer value
(42 + 0i)           float32     42.0 (with zero imaginary part) is in the set of float32 values
```

```
x                   T           x is not representable by a value of T because

0                   bool        0 is not in the set of boolean values
'a'                 string      'a' is a rune, it is not in the set of string values
1024                byte        1024 is not in the set of unsigned 8-bit integers
-1                  uint16      -1 is not in the set of unsigned 16-bit integers
1.1                 int         1.1 is not an integer value
42i                 float32     (0 + 42i) is not in the set of float32 values
1e1000              float64     1e1000 overflows to IEEE +Inf after rounding
```