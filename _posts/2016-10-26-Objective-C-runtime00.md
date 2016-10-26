
#Objective-C runtime 00-概述

OBjective-C是一门动态语言，将许多静态语言在编译和链接时期做的事情放到了运行时来处理，归于他的运行时系统Objc Runtime。Objc Runtime是一个用C和汇编写的库，使C语言具有面向对象的能力。

##动态语言的效果
对于静态语言，下面有一段C语言的代码，在main函数中先执行funA再执行funB,编译链接完成之后执行顺序就固定了。

```c
void main()
{
	funcA(); //输出funcA
	funcB(); //输出funcB
}

void funcA()
{
	print("funcA");
}

void funcB()
{
	print("funcB");
}
```

对于动态语言，比如Objective-C的下面代码，使用runtime进行方法交换之后，运行时调用funcA，确执行的是funcB

```
- (void)test {
	[self funcA]; //输出funcA
	//方法交换
	Method funcA_method = 
	class_getInstanceMethod([self class],@selector(functionA));
	Method funcB_method =
	class_getInstanceMethod([self class],@selector(functionB));
	method_exchangeImplementations(funcA_method,funcB_method)
	[self funcB]; //还是输出funcA

}

- (void)funcA
{
	NSLog(@"funcA");
}

- (void)funcB
{
	NSLog(@"funcB");
}
```
这就是动态语言的特性，在运行时可以发生改变。Objective-C的主要是根据runtime实现动态效果的。



