# Run-time panics
尝试超出范围索引数组的这种执行错误会触发运行时panic，这就相当于使用已经实现了runtime.Error接口的参数值来调用内建函数panic。该参数值的类型满足预声明接口类型error。代表不同的运行时错误条件的正确值是未指定的。

```
package runtime 

type Error interface {
	erorr
    // and perhaps other methods
}
```