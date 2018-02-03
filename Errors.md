# Errors
预先声明的error类型如下定义：
```
type error interface {
	Error() string
}
```

这是一个可以非常方便表示错误条件的接口，nil值代表没有错误。例如，从文件中读取数据的函数有可能如下定义：

`func Read(f *File, b[]byte) (n int, err error) `
