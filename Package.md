# Package
go程序是通过连接各种packages而组织起来的，而单个package是由一个或者多个源文件组织起来的，这些源文件聚集了声明的常量、类型、变量和函数，这些内容都属于这个单独的package，并且这些内容在属于这个package的所有文件中是可访问的。


## Source file organization
每一个由package子句构成的源文件定义了这些文件属于哪个一package，然后是有可能为空的一系列的包导入声明，这些导入声明表示希望使用哪些package的内容，再然后是有可能为空的一系列函数、类型、变量、常量的声明

`SourceFile		= PackageClause ";" { ImportDecl ";"} { TopLevelDecl ";" } . `

## Package clause
package子句起于每个go源文件，定义了文件所属的package

`PackageClause 	= "package" PackageName .`
`PackageName	= indentifier .`

包名不能是空白字符， 例如 `package math`

多个拥有相同包名的文件形成了一个package。形成一个package的多个文件需要在相同的目录下

## Import declarations
导入声明表示：包含导入声明的package源文件依赖被导入package的功能，并且对被导入package的可导出标识符（以大定字母开头）有使用权限。导入声明命名了一个可以被用于访问被导入package的标识符（被导入的包名），导入声明路径确定了导入哪一个package

`ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .`
`ImportSpec       = [ "." | PackageName ] ImportPath .`
`ImportPath       = string_lit .`

在导入源文件中，包名作为限定标识符被用来访问该包可被导出的标识符。该限定标识符被声明在了文件块范围中。如果包名被省略了，则使用被导入package的package子句中的包名作标识符。如果一个句号.出现在了包名的前面，则被导入package的所有可被导出的，声明在被导入package的包块范围中的标识符将会被声明在导入文件的文件块范围中，并且使用这些标识符的时候不需要限定词。

导入路径依赖于具体package的实现，但是一般情况下，导入路径是完整的文件路径名字的子串，可能和已经安装的package的仓库是对应的。

实现约束：编译器会将导入路径限制为非空字符串，这些字符串只使用属于Unicode's L, M, N, P, 和S的一般类型的字符（没人空格的图形字符），并且也会排除!"#$%&'()*,:;<=>?[\]^`{|} 这些字符和Unicode的替换字符比如U+FFFD。

假设我们已经编译好了一个package，这个package包含了package子句`package math`，包含了可导入的函数Sin，并且将该package安装在了以"lib/math"为标识的文件中。下面的内容说明了在各种导入声明后，Sin函数是如何被导入该package的文件访问的

`import		"lib/math"		math.Sin`
`import	m	"lib/math"		m.Sin`
`import . 	"lib/math"		Sin`

导入声明语句声明了在导入package和被导入package之前的依赖关系。直接或者间接的导入本身是非法的;导入package，并且也没有使用该package的任何可导出的标识符也是非法的。如果导入一个package只是为了side-effects（比如初始化），方法是使用下划线显示的用作被导入package的名字

`import _ "lib/math`




