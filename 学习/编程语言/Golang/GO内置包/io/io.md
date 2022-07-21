# io package

* [1. Constants](#1.%20Constants) (Constants)
* [2. Variables](#2.%20Variables) (Variables)
* [3. Copy](#3.%20Copy) (Func)
* [4. CopyBuffer](#4.%20CopyBuffer) (Func)
* [5. CopyN](#5.%20CopyN) (Func)
* [6. Pipe](#6.%20Pipe) (Func)
* [7. ReadAll](#7.%20ReadAll) (Func)
* [8. ReadAtLeast](#8.%20ReadAtLeast) (Func)
* [9. ReadFull](#9.%20ReadFull) (Func)
* [10. WriteString](#10.%20WriteString) (Func)
* [11. ByteReader](#11.%20ByteReader) (Type)
* [12. ByteScanner](#12.%20ByteScanner) (Type)
* [13. ByteWriter](#13.%20ByteWriter ) (Type)
* [14. Closer](#14.%20Closer) (Type)
* [15. LimitedReader](#15.%20LimitedReader) (Type)
  * [15.1. Read](#15.1.%20Read) (Func)
* [16. PipeReader](#16.%20PipeReader) (Type)
  * [16.1. Close](#16.1.%20Close) (Func)
  * [16.2. CloseWithError](#16.2.%20CloseWithError) (Func)
  * [16.3. Read](#16.3.%20Read) (Func)
* [17. PipeWriter](#17.%20PipeWriter) (Type)
  * [17.1. Close](#17.1.%20Close) (Func)
  * [17.2. CloseWithError](#17.2.%20CloseWithError) (Func)
  * [17.3. Write](#17.3.%20Write) (Func)
* [18. ReadCloser](#17.%20ReadCloser) (Type)
  * [18.1. NopCloser](#18.1.%20NopCloser) (Func)
* [19. ReadSeekCloser](#19.%20ReadSeekCloser) (Type)
* [20. ReadSeeker](#20.%20ReadSeeker) (Type)
* [21. ReadWriteCloser](#21.%20ReadWriteCloser) (Type)
* [22. ReadWriteSeeker](#22.%20ReadWriteSeeker) (Type)
* [23. ReadWriter](#23.%20ReadWriter) (Type)
* [24. Reader](#24.%20Reader) (Type)
  * [24.1. LimitReader](#24.1.%20LimitReader) (Func)
  * [24.2. MultiReader](#24.2.%20MultiReader) (Func)
  * [24.3. TeeReader](#24.3.%20TeeReader) (Func)
* [25. ReaderAt](#25.%20ReaderAt) (Type)
* [26. ReaderFrom](#26.%20ReaderFrom) (Type)
* [27. RuneReader](#27.%20RuneReader) (Type)
* [28. RuneScanner](#28.%20RuneScanner) (Type)
* [29. SectionReader](#29.%20SectionReader) (Type)
  * [29.1. NewSectionReader](#29.1.%20NewSectionReader) (Func)
  * [29.2. Read](#29.2.%20Read) (Func)
  * [29.3. ReadAt](#29.3.%20ReadAt) (Func)
  * [29.4. Seek](#29.4.%20Seek) (Func)
  * [29.5. Size](#29.5.%20Size) (Func)
* [30. Seeker](#30.%20Seeker) (Type)
* [31. StringWriter](#31.%20StringWriter) (Type)
* [32. WriteCloser](#32.%20WriteCloser) (Type)
* [33. WriteSeeker](#33.%20WriteSeeker) (Type)
* [34. Writer](#34.%20Writer) (Type)
  * [34.1. MultiWriter](#34.1.%20MultiWriter) (Func)
* [35. WriterAt](#35.%20WriterAt) (Type)
* [36. WriterTo](#36.%20WriterTo) (Type)

---

## 1. Constants

```go
const (
    SeekStart   = 0 // seek relative to the origin of the file
    SeekCurrent = 1 // seek relative to the current offset
    SeekEnd     = 2 // seek relative to the end
)
```

---

## 2. Variables

* EOF 是当没有更多可用输入时 Read 返回的错误。（读取必须返回 EOF 本身，而不是包装 EOF 的错误，因为调用者将使用 == 测试 EOF。）函数应该返回 EOF 仅表示输入的优雅结束。如果 EOF 在结构化数据流中意外发生，则相应的错误是 ErrUnexpectedEOF 或提供更多详细信息的其他错误

```go
var EOF = errors.New("EOF")
```

* ErrClosedPipe 是用于对封闭管道进行读取或写入操作的错误

```go
var ErrClosedPipe = errors.New("io: read/write on closed pipe")
```

* 当对 Read 的许多调用都未能返回任何数据或错误时，Reader 的某些客户端会返回 ErrNoProgress，这通常是 Reader 实现损坏的标志

```go
var ErrNoProgress = errors.New("multiple Read calls return no data or error")
```

* ErrShortBuffer 意味着读取需要比提供的更长的缓冲区

```go
var ErrShortBuffer = errors.New("short buffer")
```

* ErrShortWrite 表示写入接受的字节数少于请求的字节数，但未能返回显式错误

```go
var ErrShortWrite = errors.New("short write")
```

* ErrUnexpectedEOF 表示在读取固定大小的块或数据结构的过程中遇到了 EOF

```go
var ErrUnexpectedEOF = errors.New("unexpected EOF")
```

---

## 3. Copy

将副本从 src 复制到 dst，直到在 src 上达到 EOF 或发生错误。它返回复制的字节数和复制时遇到的第一个错误（如果有），如果目标文件不为空会被覆盖

成功的 Copy 返回 err == nil，而不是 err == EOF。因为 Copy 被定义为从 src 读取直到 EOF，所以它不会将 Read 的 EOF 视为要报告的错误

如果 src 实现了 WriterTo 接口，则通过调用 src.WriteTo(dst) 实现复制。否则，如果 dst 实现了 ReaderFrom 接口，则通过调用 dst.ReadFrom(src) 实现复制

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```

示例（1）：

```go
func main() {
    r := strings.NewReader("some io.Reader stream to be read\n") // strings.NewReader返回一个Reader指针类型，实现了read方法

    if _, err := io.Copy(os.Stdout, r); err != nil { // os.Stdout是标准输出，输出到终端，并且r为指针也可以作为src参数
        log.Fatal(err)
    }
    fmt.Println("finish")
}
```

输出：

```limux
some io.Reader stream to be read
finish
```

示例（2）：

```go
func func() {
    var r  *strings.Reader = strings.NewReader("简单测试一下好吗")
    var f *os.File
    var err error
    f, err = os.OpenFile("/web/code/pkg/io/test.txt", os.O_RDWR|os.O_CREATE, 0755) // 返回一个文件类型指针接口，实现了writer方法
    if err != nil {
        log.Fatal(err)
    }
    var written int64
    written, err = io.Copy(f, r)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("写入长度为：", written)
}
```

---

## 4. CopyBuffer

CopyBuffer 与 Copy 相同，只是它通过提供的缓冲区（如果需要）进行暂存，而不是分配临时缓冲区

如果 buf 为 nil，则分配一个；否则，如果它的长度为零，则 CopyBuffer 会发生混乱

如果 src 实现 WriterTo 或 dst 实现 ReaderFrom，则不会使用 buf 执行复制

```go
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error)
```

示例：

```go
func main() {
    var r1,r2 *strings.Reader
    r1 = strings.NewReader("第一个测试\n")
    fmt.Println(r1.Len())
    r2 = strings.NewReader("第二个测试\n")
    var buf []byte 
    buf = make([]byte, 8) // 分配固定内存

    var err error
    if _, err = io.CopyBuffer(os.Stdout, r1, buf); err != nil { // 两次buffer用的同一片内存地址，减少了内存动态分配和回收的次数
        log.Fatal(err)
    }

    if _, err = io.CopyBuffer(os.Stdout, r2, buf); err != nil {
        log.Fatal(err)
    }
    fmt.Println("finish")
}
```

输出：

```go
16
第一个测试
第二个测试
finish
```

---

## 5. CopyN

CopyN 将 n 个字节（或直到出错）从 src 复制到 dst

它返回复制的字节数和复制时遇到的最早错误

如果 dst 实现了 ReaderFrom 接口，则使用它实现副本

```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```

---

## 6. Pipe

Pipe 创建一个同步的内存管道。它可用于连接期望 io.Reader 的代码和期望 io.Writer 的代码

管道上的读取和写入是一对一匹配的，除非需要多个读取来消耗单个写入。也就是说，对 PipeWriter 的每次写入都会阻塞，直到它满足来自 PipeReader 的一个或多个读取，这些读取完全消耗了写入的数据。数据直接从Write复制到对应的Read（或Reads）；没有内部缓冲

彼此并行或使用 Close 调用 Read 和 Write 是安全的。对 Read 的并行调用和对 Write 的并行调用也是安全的：各个调用将按顺序进行门控

io.Pipe会返回一个reader和writer,对reader读取（或写入writer）后，进程会被锁住，直到writer有新数据流进入或关闭（或reader把数据读走）

```go
func Pipe() (*PipeReader, *PipeWriter)
```

示例（1）：

```go
func main() {
    // 没有开启协程无路读入还是写入都会造成进程死锁
    _, w := io.Pipe() // 生成管道
    fmt.Fprint(w, "some io.Reader stream to be read\n") // Fprint为格式化流写入w，阻塞进程，直到有读入
    w.Close() // 无法执行，进程死锁，报错
}
```

输出：

```linux
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select]:
io.(*pipe).write(0xc000054180, {0xc000020060, 0x21, 0x30})
        /usr/local/go/src/io/pipe.go:86 +0x212
io.(*PipeWriter).Write(0xc00006ca90?, {0xc000020060?, 0x1?, 0xc000066ef0?})
        /usr/local/go/src/io/pipe.go:165 +0x25
fmt.Fprint({0x4b5f88, 0xc00000e028}, {0xc000066f50, 0x1, 0x1})
        /usr/local/go/src/fmt/print.go:233 +0x75
main.test5()
        /web/code/pkg/io/iotest.go:22 +0x11e
main.main()
        /web/code/pkg/io/iotest.go:17 +0x17
exit status 2
```

示例（2）：

```go
func main() {
    r, w := io.Pipe() // 生成管道
    
    go func() {
        time.Sleep(5 * time.Second) // 协程内沉睡5s
        fmt.Fprint(w, "some io.Reader stream to be read\n") // Fprint为格式化流写入w，在沉睡5s后写入，阻塞进程，直到有读入
        w.Close()
    }()

    if _, err := io.Copy(os.Stdout, r); err != nil { // 阻塞进程，等待写入，5s后协程内写入，此处读入，往下执行代码
        log.Fatal(err)
    }
}
```

输出：

```go
// 此处间隔5s终端输出
some io.Reader stream to be read
```

---

## 7. ReadAll

ReadAll 从 r 读取，直到出现错误或 EOF 并返回它读取的数据

成功的调用返回 err == nil，而不是 err == EOF。因为 ReadAll 被定义为从 src 读取直到 EOF，所以它不会将 Read 的 EOF 视为要报告的错误

```go
func ReadAll(r Reader) ([]byte, error)
```

示例（1）：

```go
func test6() {
    var r *strings.Reader
    r = strings.NewReader("My favourite idle is JayChou!") // NewReader返回一个指针*Reader

    var b []byte
    var err error
    b, err = io.ReadAll(r)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("%s\n", b)
    fmt.Println(string(b))
}
```

输出：

```go
My favourite idle is JayChou!
My favourite idle is JayChou!
```

示例（2）：

```go
func main() {
    // 读取文件

    var f *os.File
    var err error
    f, err = os.OpenFile("/web/code/pkg/io/test.txt", os.O_RDWR|os.O_CREATE, 0755) // 返回*File接口，实现Read方法
    if err != nil {
        log.Fatal(err)
    }
    var b []byte
    b, err = io.ReadAll(f)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(b))
}
```

输出：

```go
// /web/code/pkg/io/test.txt 文件中的内容
简单测试一下好吗
```

---

## 8. ReadAtLeast

ReadAtLeast 从 r 读取到 buf 直到它至少读取了 min 字节

它返回复制的字节数，如果读取的字节数较少，则返回错误

仅当未读取任何字节时，该错误才是 EOF

如果在读取少于 min 字节后发生 EOF，ReadAtLeast 返回 ErrUnexpectedEOF

如果 min 大于 buf 的长度，ReadAtLeast 返回 ErrShortBuffer

```go
func ReadAtLeast(r Reader, buf []byte, min int) (n int, err error)
```

示例：

```go
func main() {
    r := strings.NewReader("some io.Reader stream to be read\n")

    buf := make([]byte, 14) // 定义一个缓冲区，减少动态分配内存和回收次数
    if _, err := io.ReadAtLeast(r, buf, 4); err != nil { // 如果buf大于min，则正确读取
        log.Fatal(err)
    }
    fmt.Printf("%s\n", buf)

    // buffer smaller than minimal read size.
    shortBuf := make([]byte, 3)
    if _, err := io.ReadAtLeast(r, shortBuf, 4); err != nil { // 如果buf小于min，会报错
        fmt.Println("error:", err)
    }

    // minimal read size bigger than io.Reader stream
    longBuf := make([]byte, 64)
    if _, err := io.ReadAtLeast(r, longBuf, 64); err != nil { // 如果还没读到min个，就表示读到了EOF，末尾了
        fmt.Println("error:", err)
    }
}
```

输出：

```linux
some io.Reader
error: short buffer
error: unexpected EOF
```

---

## 9. ReadFull

ReadFull 将 r 中的 len(buf) 个字节准确地读取到 buf 中

它返回复制的字节数，如果读取的字节数较少，则返回错误

```go
func ReadFull(r Reader, buf []byte) (n int, err error)
```

示例：

```go
func main() {
    var r *strings.Reader
    r = strings.NewReader("Hi, today si very nice.")
    var buf []byte
    var err error
    buf = make([]byte, 4) // 划分一片固定的内存区域
    if _, err = io.ReadFull(r, buf); err != nil { // 读完会有标记位置
        log.Fatal(err)
    }
    fmt.Println(string(buf))

    if _, err = io.ReadFull(r, buf); err != nil { // 继续从上一次的位置开始读
        log.Fatal(err)
    }
    fmt.Println(string(buf))

    if _, err = io.ReadFull(r, buf); err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(buf))

    var longBuf []byte
    longBuf = make([]byte, 64)
    if _, err = io.ReadFull(r, longBuf); err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(longBuf))
}
```

输出：

```linux
Hi,
toda
y si
2022/07/20 06:06:18 unexpected EOF
exit status 1
```

---

## 10. WriteString

WriteString 将字符串 s 的内容写入 w，它接受一个字节切片

```go
func WriteString(w Writer, s string) (n int, err error)
```

示例（1）：

```go
func main() {
    var err error
    if _, err = io.WriteString(os.Stdout, "Hello World\n"); err != nil {
        log.Fatal(err)
    }
}
```

输出：

```linux
Hello World
```

示例（2）：

```go
func test11() {
    var f *os.File
    var err error
    f, err = os.OpenFile("/web/code/pkg/io/test.txt", os.O_RDWR|os.O_CREATE, 0755)
    if err != nil {
        log.Fatal(err)
    }
    if _, err = io.WriteString(f, "My name is jay haha"); err != nil {
        log.Fatal(err)
    }
    fmt.Println("finish")
}
```

---

## 11. ByteReader

ByteReader 是包装 ReadByte 方法的接口

如果 ReadByte 返回错误，则没有消耗输入字节，并且返回的字节值未定义

没有实现 ByteReader 的 Reader 可以使用 bufio.NewReader 进行包装以添加此方法

```go
type ByteReader interface {
    ReadByte() (byte, error)
}
```

---

## 12. ByteScanner

ByteScanner 是将 UnreadByte 方法添加到基本 ReadByte 方法的接口

UnreadByte 导致对 ReadByte 的下一次调用返回最后读取的字节。如果最后一个操作不是对 ReadByte 的成功调用，UnreadByte 可能会返回错误，未读最后一个读取的字节（或最后一个未读字节之前的字节），或者（在支持 Seeker 接口的实现中）寻找一个字节在当前偏移之前

```go
type ByteScanner interface {
    ByteReader
    UnreadByte() error
}
```

---

## 13. ByteWriter

ByteWriter 是包装了 WriteByte 方法的接口

```go
type ByteWriter interface {
    WriteByte(c byte) error
}
```

---

## 14. Closer

Closer 是封装了基本 Close 方法的接口

第一次调用后关闭的行为是未定义的。具体的实现可能会记录他们自己的行为

```go
type Closer interface {
    Close() error
}
```

---

## 15. LimitedReader

每次限制读取多少

LimitedReader 从 R 读取数据，但将返回的数据量限制为仅 N 个字节。每次调用 Read 都会更新 N 以反映剩余的新数量。当 N <= 0 或底层 R 返回 EOF 时，Read 返回 EOF

```go
type LimitedReader struct {
    R Reader // underlying reader
    N int64  // max bytes remaining
}
```

### 15.1. Read

```go
func (l *LimitedReader) Read(p []byte) (n int, err error)
```

示例：

```go
func test12() {
    var r *strings.Reader
    r = strings.NewReader("abcdefg")

    var lR io.LimitedReader // 定义类型
    lR.R = r   // 设置阅读器
    lR.N = 10  // 设置初始未读个数
    fmt.Println("初始剩余的未读数：", lR.N)
    var b []byte
    b = make([]byte, 2) // 设置2个字符数，一次最多读两个
    _, err := lR.Read(b) // 最多读两个字符
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(b))
    fmt.Println("第一次剩余未读数：", lR.N)

    if _, err = lR.Read(b); err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(b))
    fmt.Println("第二次剩余未读数：", lR.N)
}
```

输出：

```linux
初始剩余的未读数： 10
ab
第一次剩余未读数： 8
cd
第二次剩余未读数： 6
```

---

## 16. PipeReader

PipeReader 是管道的读取部分

```go
type PipeReader struct {
    // contains filtered or unexported fields
}
```

### 16.1. Close

Close 关闭阅读器；对管道写入部分的后续写入将返回错误 ErrClosedPipe

```go
func (r *PipeReader) Close() error
```

### 16.2. CloseWithError

CloseWithError 关闭阅读器；随后对管道写入部分的写入将返回错误 err

如果前一个错误存在，CloseWithError 永远不会覆盖它，并且总是返回 nil

```go
func (r *PipeReader) CloseWithError(err error) error
```

### 16.3. Read

Read 实现了标准的 Read 接口：它从管道中读取数据，阻塞直到写入者到达或写入端关闭。如果写端因错误而关闭，则该错误将作为 err 返回；否则 err 是 EOF

```go
func (r * PipeReader ) Read(data [] byte ) (n int , err error )
```

---

### 17. PipeWriter

PipeWriter 是管道的写入部分

```go
type PipeWriter struct {
    // contains filtered or unexported fields
}
```

### 17.1 Close

Close 关闭作者；从管道的读取部分进行的后续读取将不返回任何字节和 EOF

```go
func (w *PipeWriter) Close() error
```

### 17.2 Close

CloseWithError 关闭编写器；从管道的读取部分进行的后续读取将不返回任何字节，并且如果 err 为 nil，则返回错误 err 或 EOF

如果前一个错误存在，CloseWithError 永远不会覆盖它，并且总是返回 nil

```go
func (w *PipeWriter) CloseWithError(err error) error
```

### 17.3 Close

Write 实现了标准的 Write 接口：它将数据写入管道，阻塞直到一个或多个读取器消耗完所有数据或读取端关闭。如果读取端因错误而关闭，则该 err 作为 err 返回；否则 err 是 ErrClosedPipe

```go
func (w *PipeWriter) Write(data []byte) (n int, err error)
```

---

## 18. ReadCloser

ReadCloser 是对基本 Read 和 Close 方法进行分组的接口

```go
type ReadCloser interface {
    Reader
    Closer
}
```

### 18.1. NopCloser

NopCloser 返回一个 ReadCloser，其中包含一个包装提供的 Reader r 的无操作 Close 方法

```go
func NopCloser(r Reader) ReadCloser
```

---

## 19. ReadSeekCloser

ReadSeekCloser 是对基本 Read、Seek 和 Close 方法进行分组的接口

```go
type ReadSeekCloser interface {
    Reader
    Seeker
    Closer
}
```

---

## 20. ReadSeeker

ReadSeeker 是对基本 Read 和 Seek 方法进行分组的接口

```go
type ReadSeeker interface {
    Reader
    Seeker
}
```

---

## 21. ReadWriteCloser

ReadWriteCloser 是对基本 Read、Write 和 Close 方法进行分组的接口

```go
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

---

## 22. ReadWriteSeeker

ReadWriteSeeker 是对基本 Read、Write 和 Seek 方法进行分组的接口

```go
type ReadWriteSeeker interface {
    Reader
    Writer
    Seeker
}
```

---

## 23. ReadWriter

ReadWriter 是对基本 Read 和 Write 方法进行分组的接口

```go
type ReadWriter interface {
    Reader
    Writer
}
```

---

## 24. Reader

Reader 是包装基本 Read 方法的接口

Read 最多将 len(p) 个字节读入 p。它返回读取的字节数 (0 <= n <= len(p)) 和遇到的任何错误。即使 Read 返回 n < len(p)，它也可能在调用期间使用所有 p 作为暂存空间。如果某些数据可用但 len(p) 字节不可用，则 Read 通常会返回可用的数据，而不是等待更多数据

当 Read 在成功读取 n > 0 个字节后遇到错误或文件结束条件时，它会返回读取的字节数。它可能会从同一次调用中返回（非零）错误，或者从后续调用中返回错误（并且 n == 0）。这种一般情况的一个例子是，在输入流末尾返回非零字节数的 Reader 可能会返回 err == EOF 或 err == nil。下一次读取应该返回 0，EOF

在考虑错误错误之前，调用者应始终处理返回的 n > 0 字节。这样做可以正确处理读取某些字节后发生的 I/O 错误以及允许的 EOF 行为

不鼓励 Read 的实现返回零字节计数和 nil 错误，除非 len(p) == 0。调用者应将返回 0 和 nil 视为表示没有发生任何事情；特别是它不表示EOF

实现不得保留 p

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

### 24.1. LimitReader

LimitReader 返回一个从 r 读取但在 n 个字节后以 EOF 停止的 Reader。底层实现是 *LimitedReader

```go
func LimitReader(r Reader, n int64) Reader
```

示例：

```go
func main() {
    r := strings.NewReader("some io.Reader stream to be read\n")
    lr := io.LimitReader(r, 4)

    if _, err := io.Copy(os.Stdout, lr); err != nil {
        log.Fatal(err)
    }
}
```

输出：

```linux
some
```

### 24.2. MultiReader

MultiReader 返回一个 Reader，它是提供的输入阅读器的逻辑串联

它们是按顺序读取的。一旦所有输入都返回 EOF，Read 将返回 EOF

如果任何读取器返回非零、非 EOF 错误，Read 将返回该错误

```go
func MultiReader(readers ...Reader) Reader
```

示例：

```go
func main() {
    var r1, r2, r3 *strings.Reader
    r1 = strings.NewReader("first reader\n")
    r2 = strings.NewReader("second reader\n")
    r3 = strings.NewReader("third reader\n")
    var r io.Reader
    r = io.MultiReader(r1, r2, r3)

    var err error
    if _, err = io.Copy(os.Stdout, r); err != nil {
        log.Fatal(err)
    }
}
```

输出：

```go
first reader
second reader
third reader
```

### 24.3. TeeReader

TeeReader 返回一个读取器，它将从 r 读取的内容写入 w

通过它对 r 执行的所有读取都与对 w 的相应写入匹配

没有内部缓冲 - 写入必须在读取完成之前完成

写入时遇到的任何错误都报告为读取错误

```go
func TeeReader(r Reader, w Writer) Reader
```

示例：

```go
func main() {
    var r *strings.Reader // 也可以直接 var r Reader，同一个类型可以通用
    r = strings.NewReader("this is a big part\n")

    var r1 io.Reader
    r1 = io.TeeReader(r, os.Stdout)

    if _, err := io.ReadAll(r1); err != nil { // 必须要读完，所以用 ReadAll
        log.Fatal(err)
    }
}
```

输出：

```linux
this is a big part
```

---

## 25. ReaderAt

ReadAt 是包装基本 ReadAt 方法的接口

ReadAt 将 len(p) 个字节读入 p，从底层输入源中的 offset off 开始。它返回读取的字节数 (0 <= n <= len(p)) 和遇到的任何错误

当 ReadAt 返回 n < len(p) 时，它返回一个非零错误，解释为什么没有返回更多字节。在这方面，ReadAt 比 Read 更严格

即使 ReadAt 返回 n < len(p)，它也可能在调用期间使用所有 p 作为暂存空间。如果某些数据可用但 len(p) 字节不可用，则 ReadAt 会阻塞，直到所有数据可用或发生错误。在这方面，ReadAt 与 Read 不同

如果 ReadAt 返回的 n = len(p) 个字节位于输入源的末尾，则 ReadAt 可能返回 err == EOF 或 err == nil

如果 ReadAt 正在从具有寻道偏移的输入源读取，则 ReadAt 不应影响也不受底层寻道偏移的影响

ReadAt 的客户端可以在同一个输入源上执行并行的 ReadAt 调用

实现不得保留 p

```go
type ReaderAt interface {
    ReadAt(p []byte, off int64) (n int, err error)
}
```

---

## 26. ReaderFrom

ReadFrom 是包装 ReadFrom 方法的接口

ReadFrom 从 r 读取数据，直到 EOF 或错误。返回值 n 是读取的字节数。读取期间遇到的除 EOF 之外的任何错误也会返回

如果可用，复制功能使用 ReaderFrom

```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```

---

## 27. RuneReader

RuneReader 是包装 ReadRune 方法的接口

ReadRune 读取单个编码的 Unicode 字符并返回符文及其大小（以字节为单位）。如果没有可用的字符，则会设置 err

```go
type RuneReader interface {
    ReadRune() (r rune, size int, err error)
}
```

---

## 28. RuneScanner

RuneScanner 是在基本 ReadRune 方法中添加 UnreadRune 方法的接口

UnreadRune 导致对 ReadRune 的下一次调用返回上次读取的符文。如果最后一个操作不是对 ReadRune 的成功调用，UnreadRune 可能会返回错误，未读最后一个 rune 读取（或最后一个未读 rune 之前的 rune），或者（在支持 Seeker 接口的实现中）寻找开始当前偏移之前的符文

```go
type RuneScanner interface {
    RuneReader
    UnreadRune() error
}
```

---

## 29. SectionReader

SectionReader 在底层 ReaderAt 的一部分上实现 Read、Seek 和 ReadAt

```go
type SectionReader struct {
    // contains filtered or unexported fields
}
```

### 29.1 NewSectionReader

NewSectionReader 返回一个 SectionReader，它从 r 从偏移量 off 开始读取，并在 n 个字节后以 EOF 停止

```go
func NewSectionReader(r ReaderAt, off int64, n int64) *SectionReader
```

示例（1）：

```go
func main() {
    var r *strings.Reader
    r = strings.NewReader("a boy is coming here\n") // r 内有实现 ReadAt
    var s *io.SectionReader
    s = io.NewSectionReader(r, 2, 10) // 从第2位开始，取10个

    var err error
    if _, err = io.Copy(os.Stdout, s); err != nil {
        log.Fatal(err)
    }
}
```

输出：

```linux
boy is com
```

示例（2）：

```go
func main() {
    var f *os.File
    var err error
    f, err = os.OpenFile("/web/code/pkg/io/test.txt", os.O_RDWR|os.O_CREATE, 0755)
    if err != nil {
        log.Fatal(err)
    }
    var s *io.SectionReader
    s = io.NewSectionReader(f, 0, 4)

    if _, err = io.Copy(os.Stdout, s); err != nil {
        log.Fatal(err)
    }
}
```

### 29.2 Read

```go
func (s *SectionReader) Read(p []byte) (n int, err error)
```

示例：

```go
func main() {
    var r *strings.Reader
    r = strings.NewReader("some io.Reader stream to be read\n")
    var s *io.SectionReader
    s = io.NewSectionReader(r, 0, 17) // 从0开始，读17个字符

    var err error
    var buf []byte
    buf = make([]byte, 9)
    if _, err = s.Read(buf); err != nil { // Read 就读一次，直到读完buf或者文件结束
        log.Fatal(err)
    }

    fmt.Println(string(buf))
}
```

输出：

```go
some io.R
```

### 29.3 ReadAt

```go
func (s *SectionReader) ReadAt(p []byte, off int64) (n int, err error)
```

示例：

```go
func test19() {
    r := strings.NewReader("some io.Reader stream to be read\n")
    s := io.NewSectionReader(r, 10, 17)

    buf := make([]byte, 6) // 必须读满6位
    if _, err := s.ReadAt(buf, 5); err != nil { // 这里的5和上面的10叠加，也就是从第15位开始读，17必须大于等于 5 + 6
        log.Fatal(err)
    }

    fmt.Printf("%s\n", buf)
}
```

输出：

```go
stream
```

### 29.4 Seek

```go
func (s *SectionReader) Seek(offset int64, whence int) (int64, error)
```

示例：

```go
func test20() {
    var r *strings.Reader
    r = strings.NewReader("some io.Reader stream to be read\n")
    var s *io.SectionReader
    s = io.NewSectionReader(r, 5, 18) // 从偏移量5开始，最多能读到18个

    var err error
    if _, err = io.Copy(os.Stdout, s); err != nil { // 输出1
        log.Fatal(err)
    }

    if _, err = s.Seek(1, io.SeekStart); err != nil { // io.SeekStart的值为1，表示相对于文件起始偏移量开始
        log.Fatal(err)
    }

    if _, err = io.Copy(os.Stdout, s); err != nil { // 输出2
        log.Fatal(err)
    }
}
```

输出：

```go
io.Reader stream t    // 输出1：偏移量5，从0数起，第五个开始读，往后读18个，读到t，这个t是不变的，不管后面偏移量怎么变化，也只能读到t
o.Reader stream t  // 输出2：偏移量6，总0数起，第六个开始读，往后读18个，但读到t就必须停止
```

### 29.5 Size

Size 返回节的大小（以字节为单位）

```go
func (s *SectionReader) Size() int64
```

示例：

```go
func test21() {
    var r *strings.Reader
    r = strings.NewReader("some io.Reader stream to be read\n")
    var sr *io.SectionReader
    sr = io.NewSectionReader(r, 5, 100)
    var srSize int64
    srSize = sr.Size()
    fmt.Println(srSize) // 1

    var b []byte
    b = make([]byte, 5)
    var err error
    _, err = sr.Read(b)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(b)) // 2
}
```

输出：

```go
100   // 1
io.Re // 2
```

---

## 30. Seeker

Seeker 是封装了基本 Seek 方法的接口

Seek 将下一次 Read 或 Write 的偏移量设置为 offset，根据 whence 解释：SeekStart 表示相对于文件的开头，SeekCurrent 表示相对于当前的偏移量，SeekEnd 表示相对于文件的结尾。Seek 返回相对于文件开头的新偏移量或错误（如果有）

在文件开始之前寻找偏移量是错误的。可以允许寻求任何正偏移量，但如果新偏移量超过底层对象的大小，则后续 I/O 操作的行为取决于实现

```go
type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}
```

---

## 31. StringWriter

StringWriter 是包装了 WriteString 方法的接口

```go
type StringWriter interface {
    WriteString(s string) (n int, err error)
}
```

---

## 32. WriteCloser

WriteCloser 是对基本 Write 和 Close 方法进行分组的接口

```go
type WriteCloser interface {
    Writer
    Closer
}
```

---

## 33. WriteSeeker

WriteSeeker 是对基本 Write 和 Seek 方法进行分组的接口

```go
type WriteSeeker interface {
    Writer
    Seeker
}
```

---

## 34. Writer

Writer 是封装了基本 Write 方法的接口

Write 将 len(p) 个字节从 p 写入底层数据流。它返回从 p (0 <= n <= len(p)) 写入的字节数以及遇到的任何导致写入提前停止的错误。如果返回 n < len(p)，则写入必须返回非零错误。写入不能修改切片数据，即使是临时的

实现不得保留 p

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

### 34.1. MultiWriter

MultiWriter 创建一个写入器，将其写入复制到所有提供的写入器，类似于 Unix tee(1) 命令

每次写入都会写入每个列出的作者。如果列出的写入器返回错误，则整个写入操作将停止并返回错误；它不会继续列表

```go
func MultiWriter(writers ...Writer) Writer
```

示例：

```go
func test22() {
    var r *strings.Reader
    r = strings.NewReader("some io.Reader stream to be read")

    var buf1, buf2 bytes.Buffer
    var w io.Writer
    w = io.MultiWriter(&buf1, &buf2) // 这里要指针，因为bytes.Buffer接收器需要指针，可能会对其进行修改

    var err error
    if _, err = io.Copy(w, r); err != nil {
        log.Fatal(err)
    }

    fmt.Println(buf1.String()) // 1
    fmt.Println(buf2.String()) // 2
}
```

输出：

```go
some io.Reader stream to be read // 1
some io.Reader stream to be read // 2
```

---

## 35. WriterAt

WriterAt 是包装基本 WriteAt 方法的接口

WriteAt 将 p 的 len(p) 个字节写入偏移关闭的基础数据流。它返回从 p (0 <= n <= len(p)) 写入的字节数以及遇到的任何导致写入提前停止的错误。如果 WriteAt 返回 n < len(p)，则它必须返回非 nil 错误

如果 WriteAt 正在使用查找偏移量写入目标，则 WriteAt 不应影响底层查找偏移量，也不受其影响

如果范围不重叠，WriteAt 的客户端可以在同一目标上执行并行 WriteAt 调用

实现不得保留 p

```go
type WriterAt interface {
    WriteAt(p []byte, off int64) (n int, err error)
}
```

---

## 36. WriterTo

WriterTo 是包装了 WriteTo 方法的接口

WriteTo 将数据写入 w，直到没有更多数据可写入或发生错误。返回值 n 是写入的字节数。写入期间遇到的任何错误也会返回

Copy 函数使用 WriterTo（如果可用）

```go
type WriterTo interface {
    WriteTo(w Writer) (n int64, err error)
}
```

---
