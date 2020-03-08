# 函数

`func eval(a,b int, op string) int`定义的方式和变量类似，都是函数名在前，返回值类型在后，类型是用来指定函数返回值得类型，若函数没有返回值，则不需要指定函数类型。

## 函数返回值

Go语言的函数可以返回多个值。（Python也可以），只有一个返回值的时候，可以直接指定返回值的类型，或者使用括号，如`func eval(a,b int, op string) (i int)`，但是有多个返回值的时候一定要使用括号。

```Go
func div(a, b int) (q, r int) {
	return a / b, a % b
}
```

可以直接指定函数的返回值类型，而不需要写参数

```Go
// 不写返回值参数
func div(a, b int) (int, int) {
	return a / b, a % b
}
```

其实指定返回值的参数，即这里的q，r 没有太大的意义，对调用者而言没有区别，只是起到了提示作用，便捷的地方在调用的时候编辑器会自动帮我们创建生成接受返回值的参数q，r。但是有一种情况，返回值的参数有大用：

```Go
func div(a, b int) (q, r int) {
	q = a / b
  r = a % b
  return     // 此时只要直接return即可返回值
}
```

+ 但是使用这种方式，只要函数体一长，很容易搞不清赋值情况
+ 且返回值参数一定要写

所以建议还是使用第一种方式。

可以返回多个返回值带来便利的同时也会产生问题，如必须使用相应个数的参数来接受返回值，否则会报错（Go不像Python一样可以使用`[下标]`的方式来取得值，Python返回的是元组，所以可以使用下标），这就要求我们必须使用与返回值个数匹配的参数去接受，但是Go严格的语法要求<font color="red">函数内</font>所有定义的参数都必须使用到，否则报编译错误，此时我们就可以使用`_`来接受不需要的返回值。

```Go
// 使用_来接受不需要的值
q, _ := div(a, b)
```

## 多个返回值的正常用法

<font color="red">注意：</font>不要滥用函数的多返回值，一般的常用法是用来定义一个参数用来返回错误`error`的，即一个正常的返回值，一个出现异常返回的错误值。

```Go
func eval(a, b int, op string) (int, error) {    //  f返回值可以直接类型，如这里的int和error
	switch op {
	case "+":
		return a + b, nil
	case "-":
		return a - b, nil
	case "*":
		return a * b, nil
	case "/":
		q, _ := div(a, b)
		return q, nil
	default:
    // 不使用原先的panic()函数，会中断程序的运行，而是采用fmt.Errorf()构造异常
    return 0, fmt.Errorf(  
			"unsupported operation: %s", op)
	}
}

func main(){
  if result, err := eval(3, 4, "x"); err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println(result)
	}
}
```

## 函数式编程

Go语言是函数式编程，函数式一等公民：参数，变量，返回值都可以是函数

```Go
func apply(op func(int, int) int, a, b int) int {
  // 为了获取函数名
	p := reflect.ValueOf(op).Pointer()
	opName := runtime.FuncForPC(p).Name()
	fmt.Printf("Calling function %s with args "+
		"(%d, %d)\n", opName, a, b)

	return op(a, b)
}

// 因为自带的math.Pow()方法的参数和返回值要求是float64，所以这里进行重写
func pow(a,b int) int{
  return int(math.Pow(float64(a), float64(b)))
}

func main(){
  fmt.Println(apply(pow, 3, 4))
}

// 运行结果
Calling function main.pow with args (3, 4)
81
```

除了传递手写的函数，也可以直接使用匿名函数(为省略了函数名的函数，除了函数名其他函数组件一个不能少)

```Go
func main(){
  fmt.Println(apply(
    func(a int,b int) int{
      return int(math.Pow(
        float64(a), float64(b)))
    }, 3, 4))
}

// 运行结果
Calling function main.main.func1 with args (3, 4)    
81
```

第一个main是包名，第二个main是func main()的，由于是匿名的函数，所以第三个是func1。

## 其他语言中函数的特性

因为Go的语法较为简洁，一起其他语言的特性都不包含，如默认参数，可选参数，函数重载等。

但是支持可变参数列表

```Go
func sum(numbers ...int) int {     // ...为golang的语法糖，此处表示可以接受多个int类型的参数
	s := 0
	for i := range numbers {
		s += numbers[i]
	}
	return s
}
```

## 函数语法要点回顾

+ 返回值类型写在最后
+ 可返回多个值
+ 函数作为参数
+ 没有默认参数，可选参数

