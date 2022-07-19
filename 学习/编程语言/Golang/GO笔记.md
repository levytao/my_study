# GO笔记

`1.` go语言中的int的大小是和操作系统位数相关的，如果是32位操作系统，int类型的大小就是4字节(int32)。如果是64位操作系统，int类型的大小就是8个字节(int64)。int32和int64是不同类型的类型，不能作运算

---

`2.` 结构体中参数首字母没有大写时，别的包虽然可以调用这个结构体，但是找不到这个结构体中没有首字母大写的参数

---

`3.` Printf和Sprintf都是替换字符串，Printf是直接输出到终端，Sprintf是不直接输出到终端，而是返回该字符串

---

`4.` 数名称前面的括号是Go定义这些函数将在其上运行的对象的方式

他们也被称为接收者。这里是定义他们的方法有两种。如果你想修改接收器，使用如下的指针：

```go
func (s *MyStruct) pointerMethod() { } // method on pointer
```

如果你不需要修改接收器，则可以将接收器定义为如下值：

```go
func (s MyStruct)  valueMethod()   { } // method on value
```

Go的这个例子展示了这个概念。

```go
package main
 
import "fmt"
 
type Mutatable struct {
    a int
    b int
}
 
func (m Mutatable) StayTheSame() {
    m.a = 5
    m.b = 7
}
 
func (m *Mutatable) Mutate() {
    m.a = 5
    m.b = 7
}
 
func main() {
    m := &Mutatable{0, 0}
    fmt.Println(m)
    m.StayTheSame()
    fmt.Println(m)
    m.Mutate()
    fmt.Println(m)
```

上述程序的输出是：

```go
&{0 0}
&{0 0}
&{5 7}
```

---

`5.` 即便文本相同，每次调用errors.New()函数都会返回一个不同的错误值。因此不能直接拿error类型做比较

---

`6.` 字符串[3:5]返回从第三个字符后到第五个字符，不包括3，后面的数不能大于字符串的长度，否则会报错

```go
var str string = "abcde"
fmt.Println(str[2:5]) // 返回cde
fmt.Println(str[:5])  // 返回abcde，表示从开头到第五位，包括开头
fmt.Println(str[3:])  // 返回de，表示从第三位开始到最后一位，不包括3，包括最后一位
```
