# Golang

```go
package main

import "fmt"
import "math/rand"

func main() {
    fmt.Println("Hello World")
    fmt.Println(rand.Intn(10))
}
```

## go basic

### basic data type

// 数字类型

| 名称 | 范围 |
| --- | --- |
| int | -9223372036854775808 ~ 9223372036854775807 |
| int8 | -128 ~ 127 |
| int16 | -32768 ~ 32767 |
| int32 / rune | -2147483648 ~ 2147483647 |
| int64 | -9223372036854775808 ~ 9223372036854775807 |
| uint | 0 ~ 18446744073709551615 |
| uint8 / byte | 0 ~ 255 |
| uint16 | 0 ~ 65535 |
| uint32 | 0 ~ 4294967295 |
| uint64 | 0 ~ 18446744073709551615 |


```go
bool

string



int
int8  
int16  
int32  
int64
uint 
uint8 / byte
uint16 
uint32 
uint64 
uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

### core data type

#### array

#### struct (like object in python)

```go
import "fmt"

type Person struct {
    name string
    age int
    sex string
}
p = Person{"zyl", 24, "man"}
fmt.Println(Person.name)
```

#### map (like dict in python)


```go
import "fmt"

// 声明一个map
var m map[string]Person
m = make(map[string]Person)

// 赋值
m["zyl"] = Person{"zyl", 24, "man"}

// 判断该map中是否存在该key
value, ok := m["zyl"]
if (ok) {
    fmt.Println(value)  // {"zyl", 24, "man"}
} else {
    fmt.Println("zyl not in map keys")
}

fmt.Println(m["zyl"].name)
```

#### methods && interface

go 语言没有`classes`, 然而，你可以在types上定义方法
`method`就是有一个特殊的接收参数的`function`
接收参数出现在


```go
package main

import (
    "fmt"
)

type Phone interface {
    call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
    fmt.Println("I am Nokia, I can call you!")
}

type IPhone struct {
}

func (iPhone IPhone) call() {
    fmt.Println("I am iPhone, I can call you!")
}

func main() {
    var phone Phone

    phone = new(NokiaPhone)
    phone.call()

    phone = new(IPhone)
    phone.call()

}
```

`method`有两种`receivers`, value receivers & pointer receivers
`value receivers` 相当于是将 T 拷贝了一份传递给了该方法，不会改变传入值的原始数据
`pointer receivers` 相当于将 T 的引用传递给了该方法，会改变传入值的原始数据


### pointer


```go
package main

import "fmt"

type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	q := v
	p := &v
	
	fmt.Printf("%v \n", q)
	fmt.Printf("%v \n", p)
	
	v.X = 1e9
	fmt.Printf("%v \n", q)
	fmt.Printf("%v \n", p)
}
/*
{1 2} 
&{1 2} 
{1 2} 
&{1000000000 2} 
*/
```

`q := v`  相当于将 v 的值拷贝了一份赋值给了q， 改变 v 的值类型数据不会改变 q 的值
`p := &v` 将指向 v 的指针赋值给了 p ， 改变 v 或 p 的值，对方的值也会改变


### goroutine


```go
package main

import (
  "fmt"
)

func main() {
  fmt.Println("This will happen first")

  go func() {
    fmt.Println("This will happen at some unknown time")
  }()

  fmt.Println("This will either happen second or third")

  fmt.Scanln()
  fmt.Println("done")
}
```

### 文件处理

```go
import "io/ioutil"

func main() {
    data, err := ioutil.ReadFile($filename) // 一次读取整个文件内容到内存
    
}
```


### JSON 转换 

[https://gobyexample.com/json](https://gobyexample.com/json)


### time





---

## 常用的一些内置模块

### fmt

printf主要是继承了C语言的printf的一些特性，可以进行格式化输出 
print就是一般的标准输出，但是不换行 
println和print基本没什么差别，就是最后会换行 


```go
// byte => string
func main() {
    str2 := "hello"
    data2 := []byte(str2)
    fmt.Println(data2)
    str2 = string(data2[:])
    fmt.Println(str2)
}
```

### io/ioutil


## 数据库

### redis

```go
package db

import "github.com/go-redis/redis"

var RedisClient *redis.Client

func NewClient() error {
	RedisClient = redis.NewClient(&redis.Options{
		Addr:     "10.246.71.42:16380",
		Password: "qwer1234",
		DB:       0,
	})
	_, err := RedisClient.Ping().Result()
	if err != nil {
		return err
	}
	return nil
}1
```

### mysql

### mongodb

### influxdb




---

## 参考文档

1. [最好的6个Go语言Web框架](https://blog.csdn.net/dev_csdn/article/details/78740990)

2. 



