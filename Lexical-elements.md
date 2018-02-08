# Lexical elements
## Comments
注释如同程序文档。有两种形式：
 1. 行注释以字符序列//开始，以行末终止。
 2. 通用注释以字符序列/*开始，以出现的第一个子字符序列*/终止。

注释不能被包括在rune、string类型的变量或者注释中。通用注释是不包括换行的，而是用空格代替。任何其它的注释用换行表示。

## Tokens
符号形成了go语言的字汇。有4种类型：标识符，关键字，操作符，标点符号和字面值。空白字符由空格(U+0020)，水平制表符(U+0009)，回车(U+000D)和换行(U+000A)组成，一般情况下这些空白字符会被忽略，但是如果被用来分隔符号的时候，它们便不会被忽略，否则多个符号将会合并成一个符号，下一个符号将会是形成一个有效符号的最长字符序列。

## Semicolons
正常的语法是使用分号";"来终止一行代码。go程序会忽略这些分号，并且使用现在的两种形式：
 1. 当输入被拆分成多个符号的时候，分号会立刻被自动插入到符号流中最后一个符号后，并且最后一个符号要是如下情况：：
 	- 是一个标识符
 	- 一个integer，floating-point，imaginary，rune或者string字面值
 	- break，continue，fallthrough或者return其中之一
 	- 操作符和标点符号 ++，--，)，]或者}之一
 2. 为了允许复杂语句只占据一行，在闭合符号)或者}之前的分号会被省略。

为了显示出语言的地道用法，文档中的代码例子会使用这些规则来省略分号。

## Identifiers
标识符命名程序中出现的各种实体，例如变量和类型。标识符为一个或者多个字母或者数字的序列。标识符中的第一个字符必须是字母。

`identifier = letter { letter | unicode_digit } .`

```
a
_x9
ThisVariableIsExported
αβ
```

其中有一些标识符是预定义的[predeclared]()

## Keywords
下面的关键字是预保留的系统关键字，它们不能被用作自定义标识符。

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## Operators and punctuation
下面的字符序列代表操作符(包括赋值操作符)和标点符号：

```
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

## Integer literals
整型字面值（整型数本身）是代表型常量的数字序列。可选的前缀会设置非10进制基数：0代表8进制，0x或者0X代表16进制。在16进制字面值中，字母a-f和A-F代表从10到15。

```
int_lit     = decimal_lit | octal_lit | hex_lit .
decimal_lit = ( "1" … "9" ) { decimal_digit } .
octal_lit   = "0" { octal_digit } .
hex_lit     = "0" ( "x" | "X" ) hex_digit { hex_digit } .
```

```
42
0600
0xBadFace
170141183460469231731687303715884105727
```

## Floating-point literals
浮点字面值(浮点类型数本身)是[floating-point constant]的小数表示。它包括整数部分，小数点，分数部分和一个指数部分。整数和分数部分组成了小数值;指数部分是一个e或者E字符，它前面是一个可选的符号部分。整数和分数部分的其中这一可以被省略;小数点或者指数部分的其中之一可以被省略。

```
            decimals exponent |
            "." decimals [ exponent ] .
decimals  = decimal_digit { decimal_digit } .
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals .
```

```
0.
72.40
072.40  // == 72.40
2.71828
1.e+0
6.67428e-11
1E6
.25
.12345E+5
```

## Imaginary literals
虚数字面值是[complex constant]()的虚数部分的小数表示。它由浮点字面值或者十进制整数表示，后面跟着小写字母i。

`imaginary_lit = (decimals | float_lit) "i" .`

```
0i
011i  // == 11i
0.i
2.71828i
1.e+0i
6.67428e-11i
1E6i
.25i
.12345E+5i
```
## Rune literals
rune字面值(rune类型c中的char)代表一个[rune constant]()，一个表示unicode字符的整数值。rune字面值由一个或者多个字符表示，并被写在单引号内。除了换行和未转义单引号外，其它的字符都可以出现在单引号内。单引号字符代表了字符本身的unicode值，而多字符序列以反斜杠开始并以各种格式出现。

单引号内的最简单形式代表了单个字符;因为go的源代码文本是unicode形式的，并被编码为utf-8格式，所以多个utf-8编码的字节可以表示为一个整数值。例如，字面值'a'占用了一个字节，字面值为a，unicode编码为U+0061，值为0x61，而'ä'占用两人字节(0xc3 0xa4)表示字面值'ä',unicode编码为U+00E4，值为0xe4。

多个反斜杠允许任意值被编码作ascii。有4种方法将整数值表示为一个数字常量:\x后跟两个16进制数字;\u后跟4个16进制数字;\u后跟8个16进制数字,然后一个普通的反斜杠\后跟三个8进制数字。在上面的每一种情况下，字面值为以对应基数的数字形式表示的值。

尽管这些表示方法都可以表示为一个整数，但是它们有不同的有效值范围。8进制的转义必须表示一个从0到255的值。16进制的转义必须通过构造才能满足一个整数的表示。转义符号\u和\U表示unicode编码，因此，某些值出现在其中是无效的，尤其是那些在0x10FFFF之上和替代的值。

在反斜杠之后，单个字符的转义代表了特殊的值：

```
\a   U+0007 alert or bell
\b   U+0008 backspace
\f   U+000C form feed
\n   U+000A line feed or newline
\r   U+000D carriage return
\t   U+0009 horizontal tab
\v   U+000b vertical tab
\\   U+005c backslash
\'   U+0027 single quote  (valid escape only within rune literals)
\"   U+0022 double quote  (valid escape only within string literals)
```

其它所有在rune字面值中以反斜杠开始的序列都是非法的。

```
rune_lit         = "'" ( unicode_value | byte_value ) "'" .
unicode_value    = unicode_char | little_u_value | big_u_value | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .
little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                           hex_digit hex_digit hex_digit hex_digit .
escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
```

```
'a'
'ä'
'本'
'\t'
'\000'
'\007'
'\377'
'\x07'
'\xff'
'\u12e4'
'\U00101234'
'\''         // rune literal containing single quote character
'aa'         // illegal: too many characters
'\xa'        // illegal: too few hexadecimal digits
'\0'         // illegal: too few octal digits
'\uDFFF'     // illegal: surrogate half
'\U00110000' // illegal: invalid Unicode code point
```

## String literals
字符串字面值表示由字符序列拼接成的[string constant]()。有两种形式：原生字符串字面值和解析字符串字面值。

原生字符串字面值是处于反引号之间的字符序列，`foo`。除了反引号外，其它任意字符都可以出现在其内。原生字符串字面值是由反引号之前未解析的(隐式的为utf-8编码)字符组成的字符串;特别的，反斜杠在其中没有特殊意思，并且原生字符串字面值可以包括换行。在原生字符串字面值中的回车字符'\r'将会被忽略。

解析字符串字面值是位于双引号单的字符串，例如"bar"。在双引号内，除了换行和未转义的双引号外，其它所有的字符都可以出现在其中。双引号之前的文本形成了该字面值，并且带反斜杠的内容会解析成他们在[rune literals]中表示的内容(\'是非法的而\"是合法的)，
并且拥有和[rune literals]()相同的约束。三个数字的八进制(\nnn)和两个数字的16进制(\xnn)转义代表结果字符串的单个字节;所有其它的转义(可能是多字节)代表单个字符的utf-8编码。因此在一个字符串中，\377和\xFF代表一个字节的值0xFF=255，而 ÿ \u00FF，\u000000FF和\xc3\xbf代表字符U+00FF以utf-8形式编码的两个字节。

```
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { unicode_char | newline } "`" .
interpreted_string_lit = `"` { unicode_value | byte_value } `"` .

```

```
`abc`                // same as "abc"
`\n
\n`                  // same as "\\n\n\\n"
"\n"
"\""                 // same as `"`
"Hello, world!\n"
"日本語"
"\u65e5本\U00008a9e"
"\xff\u00FF"
"\uD800"             // illegal: surrogate half
"\U00110000"         // illegal: invalid Unicode code point

```

下面的例子代表了相同的字符串：

```

"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

如果源代码将一个字符表示为两个unicode字符，例如涉及到字母和方言的组合形式，如果放置到[rune literal]()中将会是错误的，而如果放置到字符串字面值中，将会表示为两个个unicode字符。