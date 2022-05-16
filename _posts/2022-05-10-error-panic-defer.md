---

layout: single  
title: "Error, panic and defer"  
date: 2022-05-10 13:09:00 +0800   
categories: Go Go-Base

---

* error类型其实是一个接口类型，也是一个 Go 语言的内建类型。在这个接口类型的声明中只包含了一个方法Error。Error方法不接受任何参数，但是会返回一个string类型的结果。它的作用是返回错误信息的字符串表示形式。
* errors.New函数返回给我们一个包含了这个错误信息的error类型值。该值的静态类型当然是error，而动态类型则是一个在errors包中的，包级私有的类型*errorString。
* error类型值的Error方法就相当于其他类型值的String方法。
* 
* fmt.Printf函数如果发现被打印的值是一个error类型的值，那么就会去调用它的Error方法。fmt包中的这类打印函数都是这么做的。
* 对于具体错误的判断，Go 语言中都有哪些惯用法？
	*  对于类型在已知范围内的一系列错误值，一般使用类型断言表达式或类型switch语句来判断；
	*  对于已有相应变量且类型相同的一系列错误值，一般直接使用判等操作来判断；
	*  对于没有相应变量且类型未知的一系列错误值，只能使用其错误信息的字符串表示形式来做判断。
* 一般错误的指针类型都是error接口的实现类型，同时它们也都包含了一个名叫Err，类型为error接口类型的代表潜在错误的字段。
* 立体的错误类型体系：用类型建立起树形结构的错误体系，用统一字段建立起可追根溯源的链式错误关联
* 扁平的错误值列表：若干个名称不同但类型相同的错误值集合。（预先创建一些代表已知错误的错误值时候可以使用这种方式，）
* 由于error是接口类型，所以通过errors.New函数生成的错误值只能被赋给变量，而不能赋给常量，又由于这些代表错误的变量需要给包外代码使用，所以其访问权限只能是公开的。

	```
	var errClosed = errors.New("file already closed") // 变量名首字母小写，以防止来自包外的修改。

	func ErrClosed() error { // 函数名首字母大写，以便来自包外的访问。
 		 return errClosed // 总是返回同一个内部的错误值。
	}
	
	```
	这样就可以完美避免掉包外代码对原有错误值的恶意/意外修改了，同时还不会影响到错误值的获取和判断。
* defer语句和recover函数可以处理恢复panic，但一些runtime抛出来的panic是不可恢复的，因为问题很严重必须整改代码才行。
* defer 关键字对应的 runtime.deferproc 会将延迟调用函数与调用方所在 Goroutine 进行关联。所以当程序发生崩溃时只会调用当前 Goroutine 的延迟调用函数。(即goroutine子线程的panic不会传播到主线程)。
* defer函数调用的执行顺序与它们分别所属的defer语句的出现顺序（更严谨地说，是执行顺序）完全相反。在defer语句每次执行的时候，Go 语言会把它携带的defer函数及其参数值另行存储到一个链表中。它是先进后出（FILO）的，相当于一个栈。
* 某个goroutine中的panic是不可能由别的goroutine中的recover恢复的。或者说，一个goroutine中的panic只能由自己例程中的recover恢复。
* defer在压栈前先求出函数参数的值，比如：

	```
	func test() {
		var a, b = 1, 1
		defer func(flag int) {
			fmt.Println(flag)
		}(a + b)
	
		a = 2
		b = 2
	}
	输出 2，而不是4
	```
* defer 函数的执行既不是在 return 之后也不是在 return 之前，而是一条go语言的 return 语句包含了对 defer 函数的调用：先保存返回值到栈上；再调用defer函数。