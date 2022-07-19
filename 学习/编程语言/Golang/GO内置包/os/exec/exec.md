# exec包

* [1. ErrNotFound](#1.%20ErrNotFound) (Var)
* [2. LookPath](#2.%20LookPath) (Func)
* [3. Cmd](#3.%20cmd) (Type)
  * [3.1. Command](#3.1.%20Command) (Func)
  * [3.2. CommandContext](#3.2.%20CommandContext) (Func)
  * [3.3. Output](#3.3.%20Output) (Func)
  * [3.4. CombinedOutput](#3.4.%20CombinedOutput) (Func)
  * [3.5. Run](#3.5.%20Run) (Func)
  * [3.6. Start](#3.6.%20Start) (Func)
  * [3.7. StderrPipe](#3.7.%20StderrPipe) (Func)
  * [3.8. StdinPipe](#3.8.%20StdinPipe) (Func)
  * [3.9. StdoutPipe](#3.9.%20StdoutPipe) (Func)
  * [3.10. String](#3.10.%20String) (Func)
  * [3.11. Wait](#3.11.%20Wait) (Func)
* [4. Error](#4.%20Error) (Type)
  * [4.1. Error](#4.1.%20Error) (Func)
  * [4.2. Unwrap](#4.2.%20Unwrap) (Func)
* [5. ExitError](#5.%20ExitError) (Type)
  * [5.1. Error](#5.1.%20Error) (Func)

---

## 1. ErrNotFound

ErrNotFound 是路径搜索未能找到可执行文件时产生的错误

```go
var ErrNotFound = errors.New("executable file not found in $PATH")
```

示例：

```go
func main() {
    fmt.Println(exec.ErrNotFound)
}
```

输出：

```linux
executable file not found in $PATH
```

---

## 2. LookPath

LookPath 在由 PATH 环境变量命名的目录中搜索可执行的命名文件。

如果文件包含斜杠，则直接尝试，不参考 PATH。

结果可能是绝对路径或相对于当前目录的路径

```go
func LookPath(file string) (string, error)
```

示例：

```go
func main() {
    path, err := exec.LookPath("git") // 若file参数没有"/"，则以系统环境变量PATH的所有目录为根搜索file可执行文件
    if err != nil {
        log.Fatal("installing fortune is in your future")
    }
    fmt.Printf("fortune is available at %s\n", path)
}
```

输出：

```linux
fortune is available at /usr/local/git/bin/git
```

---

## 3. Cmd

Cmd 表示正在准备或运行的外部命令

Cmd 在调用其 Run、Output 或 CombinedOutput 方法后不能被重用

```go
type Cmd struct {
    // Path is the path of the command to run.
    //
    // This is the only field that must be set to a non-zero
    // value. If Path is relative, it is evaluated relative
    // to Dir.
    Path string

    // Args holds command line arguments, including the command as Args[0].
    // If the Args field is empty or nil, Run uses {Path}.
    //
    // In typical use, both Path and Args are set by calling Command.
    Args []string

    // Env specifies the environment of the process.
    // Each entry is of the form "key=value".
    // If Env is nil, the new process uses the current process's
    // environment.
    // If Env contains duplicate environment keys, only the last
    // value in the slice for each duplicate key is used.
    // As a special case on Windows, SYSTEMROOT is always added if
    // missing and not explicitly set to the empty string.
    Env []string

    // Dir specifies the working directory of the command.
    // If Dir is the empty string, Run runs the command in the
    // calling process's current directory.
    Dir string

    // Stdin specifies the process's standard input.
    //
    // If Stdin is nil, the process reads from the null device (os.DevNull).
    //
    // If Stdin is an *os.File, the process's standard input is connected
    // directly to that file.
    //
    // Otherwise, during the execution of the command a separate
    // goroutine reads from Stdin and delivers that data to the command
    // over a pipe. In this case, Wait does not complete until the goroutine
    // stops copying, either because it has reached the end of Stdin
    // (EOF or a read error) or because writing to the pipe returned an error.
    Stdin io.Reader

    // Stdout and Stderr specify the process's standard output and error.
    //
    // If either is nil, Run connects the corresponding file descriptor
    // to the null device (os.DevNull).
    //
    // If either is an *os.File, the corresponding output from the process
    // is connected directly to that file.
    //
    // Otherwise, during the execution of the command a separate goroutine
    // reads from the process over a pipe and delivers that data to the
    // corresponding Writer. In this case, Wait does not complete until the
    // goroutine reaches EOF or encounters an error.
    //
    // If Stdout and Stderr are the same writer, and have a type that can
    // be compared with ==, at most one goroutine at a time will call Write.
    Stdout io.Writer
    Stderr io.Writer

    // ExtraFiles specifies additional open files to be inherited by the
    // new process. It does not include standard input, standard output, or
    // standard error. If non-nil, entry i becomes file descriptor 3+i.
    //
    // ExtraFiles is not supported on Windows.
    ExtraFiles []*os.File

    // SysProcAttr holds optional, operating system-specific attributes.
    // Run passes it to os.StartProcess as the os.ProcAttr's Sys field.
    SysProcAttr *syscall.SysProcAttr

    // Process is the underlying process, once started.
    Process *os.Process

    // ProcessState contains information about an exited process,
    // available after a call to Wait or Run.
    ProcessState *os.ProcessState
    // contains filtered or unexported fields
}
```

### 3.1. Command

命令返回 Cmd 结构以执行具有给定参数的命名程序

它仅在返回的结构中设置 Path 和 Args

如果 name 不包含路径分隔符，Command 会尽可能使用 LookPath 将 name 解析为完整路径（即系统环境变量PATH）。否则它直接使用名称作为路径

运行之后返回一个 *Cmd 指针

参数：

* name : 终端执行的命令（可执行文件），若没有 "/" 则默认在/usr/bin/*中找name命令，否则以整个name为命令

* arg : 命令后面的参数，如 ls /web，其中 /web 就是参数

```go
func Command(name string, arg ...string) *Cmd
```

### 3.2. CommandContext

CommandContext 与 Command 类似，但包含一个上下文

```go
func CommandContext(ctx context.Context, name string, arg ...string) *Cmd
```

### 3.3. Output

输出运行命令并返回其标准输出

任何返回的错误通常都是 *ExitError 类型

如果 c.Stderr 为 nil，则输出填充 ExitError.Stderr

```go
func (c *Cmd) Output() ([]byte, error)
```

示例：

```go
func main() {
    var cmd *exec.Cmd
    cmd = exec.Command("ls", "/web")
    var out []byte
    var err error
    out, err = cmd.Output()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(out))
}
```

输出：

```linux
code
etcd-v3.5.4-linux-amd64.tar.gz
ftp-0.17-67.el7.x86_64.rpm
gin@v1.8.0
go1.18.2.linux-amd64.tar.gz
...
```

### 3.4. CombinedOutput

CombinedOutput 运行命令并返回其组合的`标准输出`和`标准错误`

```go
func (c *Cmd) CombinedOutput() ([]byte, error)
```

### 3.5. Run

Run 启动指定的命令并等待它完成

如果命令运行、复制标准输入、标准输出和标准错误没有问题，并且以零退出状态退出，则返回的错误为 nil

如果命令启动但未成功完成，则错误类型为 *ExitError。对于其他情况，可能会返回其他错误类型

如果调用 goroutine 已经用 runtime.LockOSThread 锁定了操作系统线程并修改了任何可继承的操作系统级线程状态（例如，Linux 或 Plan 9 名称空间），则新进程将继承调用者的线程状态

```go
func (c *Cmd) Run() error
```

示例：

```go
func main() {
    cmd := exec.Command("sleep", "1")
    log.Printf("Running command and waiting for it to finish...")
    err := cmd.Run() // 等待命令执行完毕
    log.Printf("Command finished with error: %v", err)
}
```

输出：

```go
2022/07/19 06:47:36 Running command and waiting for it to finish...
//间隔一秒
2022/07/19 06:47:37 Command finished with error: <nil>
```

### 3.6. Start

Start 启动指定的命令，但不等待它完成

如果 Start 返回成功，将设置 c.Process 字段

一旦命令退出，Wait 方法将返回退出代码并释放相关资源

```go
func (c *Cmd) Start() error
```

示例：

```go
func main() {
    var cmd *exec.Cmd
    cmd = exec.Command("sleep", "5")
    var err error
    err = cmd.Start() // 不等待，直接往后执行
    if err != nil {
        log.Fatal(err)
    }
    log.Println("Waiting for command to finish...")
    err = cmd.Wait() // Wait方法等待cmd命令执行完成退出
    log.Printf("Command finished with error: %v", err)
}
```

输出：

```go
2022/07/19 07:02:22 Waiting for command to finish... // 即时输出
// 间隔5秒
2022/07/19 07:02:27 Command finished with error: <nil>
```

### 3.7. StderrPipe

StderrPipe 返回一个管道，该管道将在命令启动时连接到命令的标准错误

Wait 会在看到命令退出后关闭管道，所以大多数调用者不需要自己关闭管道

因此，在管道的所有读取完成之前调用 Wait 是不正确的

同理，在使用 StderrPipe 时使用 Run 是不正确的

```go
func (c *Cmd) StderrPipe() (io.ReadCloser, error)
```

示例：

```go
func func() {
    cmd := exec.Command("ls", "/mm") // 输入一个不存在的目录，产生报错
    stderr, err := cmd.StderrPipe() // 返回一个连接命令标准错误的管道，并在命令产生错误后会输入到管道内
    if err != nil {
        log.Fatal(err)
    }

    if err := cmd.Start(); err != nil {
        log.Fatal(err)
    }

    slurp, _ := io.ReadAll(stderr) // 读取所有的管道内错误信息
    fmt.Printf("错误是：%s\n", slurp)

    if err := cmd.Wait(); err != nil { // 命令结束自动关闭管道
        log.Fatal(err)
    }
}
```

### 3.8. StdinPipe

StdinPipe 返回一个管道，该管道将在命令启动时连接到命令的标准输入

Wait 看到命令退出后，管道将自动关闭

调用者只需调用 Close 即可强制管道更快关闭。例如，如果正在运行的命令在标准输入关闭之前不会退出，则调用者必须关闭管道

```go
func (c *Cmd) StdinPipe() (io.WriteCloser, error)
```

示例：

```go
func main() {
    cmd := exec.Command("cat")
    stdin, err := cmd.StdinPipe()
    if err != nil {
        log.Fatal(err)
    }

    go func() {
        defer stdin.Close()
        io.WriteString(stdin, "values written to stdin are passed to cmd's standard input")
    }()

    out, err := cmd.CombinedOutput()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("%s\n", out)
}
```

输出：

```linux
values written to stdin are passed to cmd's standard input
```

### 3.9. StdoutPipe

StdoutPipe 返回一个管道，该管道将在命令启动时连接到命令的标准输出

Wait 会在看到命令退出后关闭管道，所以大多数调用者不需要自己关闭管道

因此，在管道的所有读取完成之前调用 Wait 是不正确的

同理，在使用 StdoutPipe 时调用 Run 是不正确的

```go
func (c *Cmd) StdoutPipe() (io.ReadCloser, error)
```

示例：

```go
    cmd := exec.Command("echo", "-n", `{"Name": "Bob", "Age": 32}`)
    stdout, err := cmd.StdoutPipe()
    if err != nil {
        log.Fatal(err)
    }
    if err := cmd.Start(); err != nil {
        log.Fatal(err)
    }
    var person struct {
        Name string
        Age  int
    }
    if err := json.NewDecoder(stdout).Decode(&person); err != nil {
        log.Fatal(err)
    }
    if err := cmd.Wait(); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s is %d years old\n", person.Name, person.Age)
```

输出：

```linux
Bob is 32 years old
```

### 3.10. String

字符串返回 c 的人类可读描述

它仅用于调试

```go
func (c *Cmd) String() string
```

### 3.11. Wait

Wait 等待命令退出并等待任何复制到 stdin 或从 stdout 或 stderr 复制完成

该命令必须已由 Start 启动

如果命令运行、复制标准输入、标准输出和标准错误没有问题，并且以零退出状态退出，则返回的错误为 nil

如果命令运行失败或未成功完成，则错误类型为 *ExitError。对于 I/O 问题，可能会返回其他错误类型

如果 c.Stdin、c.Stdout 或 c.Stderr 中的任何一个不是 *os.File，Wait 也会等待相应的 I/O 循环复制到进程或从进程复制完成

等待释放与 Cmd 关联的所有资源

```go
func (c *Cmd) Wait() error
```

---

## 4. Error

当 LookPath 无法将文件分类为可执行文件时，将返回错误

```go
type Error struct {
    // Name is the file name for which the error occurred.
    Name string
    // Err is the underlying error.
    Err error
}
```

## 4.1. Error

```go
func (e *Error) Error() string
```

## 4.2. Unwrap

```go
func (e *Error) Unwrap() error
```

---

## 5. ExitError

ExitError 通过命令报告不成功的退出

```go
type ExitError struct {
    *os.ProcessState

    // Stderr holds a subset of the standard error output from the
    // Cmd.Output method if standard error was not otherwise being
    // collected.
    //
    // If the error output is long, Stderr may contain only a prefix
    // and suffix of the output, with the middle replaced with
    // text about the number of omitted bytes.
    //
    // Stderr is provided for debugging, for inclusion in error messages.
    // Users with other needs should redirect Cmd.Stderr as needed.
    Stderr []byte // Go 1.6
}
```

## 5.1. Error

```go
func (e *ExitError) Error() string
```