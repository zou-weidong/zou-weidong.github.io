---
title: "语法"
linkTitle: "语法"
weight: 1
description: >
  golang 语法
---

{{% pageinfo %}}

{{% /pageinfo %}}


## 变量声明
指定变量类型，如果没有初始化，则变量默认为零值。
如果变量已经使用 var 声明过了，再使用 := 声明变量，就产生编译错误。


默认的零值：
- 数值类型为0
- 布尔类型为 false
- 字符串为 "" (空字符串)
- 以下几种类型为nil
    - var a *int
    - var a []int
    - var a map[string] int
    - var a chan int
    - var a func(string) int
    - var a error // error 是接口


## 值类型和引用类型
像 int  float bool string 这些类型都是值类型，使用这些类型的变量直接指向存在内存中的值。
当使用等号 = 将一个变量的值赋给另一个变量时，如： j = i ，实际上是在内存中将 i 的值进行了拷贝。

可以通过 &i 来获取 i 的内存地址，值类型的变量的值存储在堆中。

通常更复杂的数据会使用多个字，这些数据一般使用引用类型保存。
一个引用类型的变量 r1 存储的是 r1 的值所在的内存地址，或者内存地址中第一个字所在的位置。
这个内存地址被称为指针，这个指针实际上也是被存在另外的某一个值中。



## 常量
常量是一个简单值的标识符，在程序运行时，不会被修改的值。
常量中的数据类型只能是 布尔 数字型（整数型、浮点和复数）和字符串。

定义的格式：

> const identifier [type] = value

可以省略 [type] ，编译器可以根据变量的值来推断其类型。
多个相同类型的声明可以简写为：

> const c_name1, c_name2 = value1, value2

## 运算符
比较重要的两个：
- & 返回变量存储地址  
- * 指针变量

如:

```
a := 4
b = &a
fmt.Println(b)  # 0xc00000a0c8
fmt.Println(*b) # 4
```

可以记忆为： 星号是变量

## 函数
函数参数，有两种方式来传递：
- 值传递：在调用函数时将实际参数复制一份传递到函数中，这样在函数中对参数进行修改，不会影响到实际参数。
- 引用传递：引用传递时指将实际参数的地址传递到函数中，在函数中对参数进行修改，将影响到实际参数。


函数用法：
- 函数作为另一个函数的实参
- 闭包
- 方法

## 数组
声明需要指定元素类型以及元素个数，在声明的时候指定大小，每个元素会根据数据类型进行默认初始化，对于整数类型，初始化为0。
> var arrayName [size]dataType

还可以使用初始化列表来初始化数组的元素：
> numbers := [5] int{1,2,3,4,5}


还得理解：
- 多维数组
- 向函数传递数组

## 指针
指针变量指向了一个值的内存地址。类似于变量和常量，在使用指针前需要声明指针，声明格式如下：
> var var_name *var-type

eg:
```
var ip *int
var fp *float32
```

指针使用流程：
1. 定义指针变量
2. 为指针变量赋值
3. 访问指针变量中指向地址的值

在指针类型前面加上 * 号来获取指针所指向的内容。

```go
var a int= 20   /* 声明实际变量 */
var ip *int        /* 声明指针变量 */
ip = &a  /* 指针变量的存储地址 */
```

## 空指针
当一个指针被定义后没有分配到任何变量时，它的值是nil。nil指针也被称为空指针。 一个空指针通常缩写为 ptr
> var ptr *int

空指针判断：
- if(ptr != nil) 
- if(ptr == nil)


## 结构体
数组存储同一类型的数据，结构体中可以为不同项定义不同的数据类型。
```go
type struct_type struct {
  age int
}
```

结构体指针：
```
var struct_pointer *Books
struct_pointer = &Book
```

## 切片
切片是对数组的抽象。

Go 数组的长度不可改变，在特定场景中不适用。相比切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大。

定义切片：
> var identifier []type

切片不需要说明长度，或者使用make()函数来创建切片：

```go
var slice1 []type = make([]type, len)
slice2 := make([]type, len)
slice3 := make([]type,len, capacity) #容量是可选参数
```

初始化切片，其中cap=len=3：
> s := [] int {1,2,3}

切片在未初始化之前默认是nil，长度为0。


隔断：
通过设置下限以及上限来设置截取切片。


append()和copy()函数：
如果想增加切片的容量，创建一个新的更大的切片把原切片内容都拷贝过来。

## 范围
Go 语言中 range 关键字在于 for 循环中迭代数组、切片、通道和集合的元素，在数组和切片中返回元素的索引和索引对应的值，在集合中返回 key - value 对。
```go
for key,value := range oldMap {
  newMapp[key] = value
}
```
在上述代码中，key和 value 都是可以省略的。

```go
for key := range oldMap #省略 val

for ——，val：= range oldMap # 省略 key

for key,_ := range oldMap # 省略val

```

## Map 集合

无序的键值对的集合。通过 key 快速检索数据，key 类似于索引，指向数据的流。

### 定义 Map：
使用内建函数 make 或使用 map 关键字来定义 Map
> map_variable := make(map[keyType]ValueType, initialCapacity)

当 Map 中的键值对数量达到容量时，Map会自动扩容。
如果不指定 initialCapacity，Go 语言会根据实际情况选择一个合适的值。

```go
# 空 map
map1 := make(map[string]int)

# 初始化容量是10
map2 := make(map[string]int,10)

# 字面量创建 Map
map3 := map[string]int{
  "a": 1,
  "b": 2,
  "c": 3,
}

#  获取元素
v1 := map3["a"]
v2, ok := map3["b"]

# 修改元素
m["a"] =5

# 获取Map的长度
len := len(m)

# 删除
delete(m,"a")

# 遍历

for k ,v := range m{
  fmt.Printf("key=%s, value=%d\n",k,v)
}

```


## 类型转换
类型转换的基本格式如下：
> type_name(expression)

type_name 为类型， expression 为表达式。


```go
# 整数转浮点型
var a int = 10;
var b float64 = float64(a)


# 字符串类型转换
var str string = "10"
var num int
num,_ = strconv.Atoi(str) # 第二个是可能的错误


# 整数转字符串
num := 123
str := strconv.Itoa(num)

# 字符串转为浮点数
str := "3.14"
num,err := strconv.ParseFloat(str,64)


# 接口类型转换  value.(Type)
var i interface{} = "hi"
str,ok := i.(string)
```

## 错误处理
error类型是一个接口类型
```go
type error interface {
  Error() string
}
```

函数通常在最后的返回值中返回错误信息。使用 errors.New 可返回一个错误信息。

```go
error.New("this is a error.")
```


## 并发
通过 go 关键字来开启 goroutine 即可。
goroutine 是轻量级线程，调度是由Golang 运行时进行管理的。
> go 函数名(参数列表)


### 通道
通道是用来传递数据的一个数据结构。
通道可以用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 <- 用于指定通道的方向用于发送和接收，如果未指定方向则是双向通道。
```
ch <- v      // 把 v 发送给通道 ch
v := <- ch  // 从 ch 接收数据，把值赋给 v
```

声明通道：
> ch := make(chan int)


默认情况下，通道是不带缓冲区的。发送端发送数据，同时必须有接收端相应的接收数据。







