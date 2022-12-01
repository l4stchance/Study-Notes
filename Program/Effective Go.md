# 格式化

1. 我们使用制表符 (tab) 缩进， gofmt 默认也是用它，在你认为有必要时再使用空格

2. 比起 C 和 Java ，Go 所需要的括号更少

# 代码注释

1. Go 语言支持 C 风格的块注释 `/* */` 和 C++ 风格的行注释 `//` 。行注释更为常用，而块注释则主要用作包的注释

    这些注释的风格决定了 godoc 生成的文档质量

2. godoc 对源码进行处理，并提取包中的文档内容，出现在顶级声明之前，且与该声明之间没有空行的注释。

    ```go
    /*
    regexp 包为正则表达式实现了一个简单的库。
    
    该库接受的正则表达式语法为：
    
        regexp:
            concatenation { '|' concatenation }
        concatenation:
            { closure }
        closure:
            term [ '*' | '+' | '?' ]
        term:
            '^'
            '$'
            '.'
            character
            '[' [ '^' ] character-ranges ']'
            '(' regexp ')'
    */
    package regexp
    ```

    每个包都应包含一段**包注释**，即放置在包子句前的一个块注释。若某个包比较简单，则可使用行注释的风格来书写包注释

    对于包含多个文件的包， 包注释只需出现在其中的任一文件中即可。包注释应在整体上对该包进行介绍，并提供包的相关信息。

3. 在包中，任何顶级声明前面的注释都将作为该声明的**文档注释**。 在程序中，每个可导出（首字母大写）的名称都应该有文档注释。

    ```go
    // Compile 用于解析正则表达式并返回，如果成
    // 功，则 Regexp 对象就可用于匹配所针对的文本。
    func Compile(str string) (*Regexp, error) {}
    ```

4. Go 的声明语法允许成组声明。单个文档注释应介绍一组相关的常量或变量。 由于是整体声明，这种注释往往较为笼统。

    ```go
    // 表达式解析失败后返回错误代码
    var (
        ErrInternal      = errors.New("regexp: internal error")
        ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
        ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
        ...
    )
    ```

# 命名规范

1. 某个名称在包外是否可见，取决于其首个字符是否为大写字母

2. 按照惯例， 包应当以小写的单个单词来命名，且不应使用下划线或驼峰记法。

3. 包名应为其源码目录的基本名称。在 src/encoding/base64 中的包应作为 "encoding/base64" 导入，其包名应为 base64， 而非 encoding_base64 或 encodingBase64。

4. 同样，用于创建 ring.Ring 的新实例的函数（这就是 Go 中的构造函数）一般会称之为 NewRing，但由于 Ring 是该包所导出的唯一类型，且该包也叫 ring，因此它可以只叫做 New，它跟在包的后面，就像 ring.New。使用包结构可以帮助你选择好的名称。

5. 获取器和设置器

    ```go
    owner := obj.Owner()
    if owner != user {
        obj.SetOwner(user)
    }
    ```

6. 按照约定，只包含一个方法的接口应当以该方法的名称加上 - er 后缀来命名，如 `Reader`、`Writer`

7. Go 中的约定是使用 `MixedCaps` 或 `mixedCaps` 而不是下划线来编写多个单词组成的命名。

# 分号

1. 通常 Go 程序只在诸如 `for` 循环子句这样的地方使用分号， 以此来将初始化器、条件及增量元素分开。

# 控制结构

1. 由于 `if` 和 `switch` 可接受初始化语句， 因此用它们来设置局部变量十分常见。

    ```go
    if err := file.Chmod(0664); err != nil {
        log.Print(err)
        return err
    }
    ```

2. 对于字符串，`range` 能够提供更多便利。它能通过解析 UTF-8， 将每个独立的 Unicode 码点分离出来。错误的编码将占用一个字节，并以符文 `U+FFFD` 来代替。

3. `rune` 是 Go 对单个 Unicode 码点的成称谓

4. Go 没有逗号操作符，而 `++` 和 `--` 为语句而非表达式。

5. `switch` 并不会自动下溯，但 `case` 可通过逗号分隔来列举相同的处理条件。

    ```go
    func shouldEscape(c byte) bool {
        switch c {
        case ' ', '?', '&', '=', '#', '+', '%':
            return true
        }
        return false
    }
    ```

6. `switch` 也可用于判断接口变量的动态类型。

    ```go
    var t interface{}
    t = functionOfSomeType()
    switch t := t.(type) {
    default:
        fmt.Printf("unexpected type %T\n", t)     // %T 打印任何类型的 t
    case bool:
        fmt.Printf("boolean %t\n", t)             // t 是 bool 类型
    case int:
        fmt.Printf("integer %d\n", t)             // t 是 int 类型
    case *bool:
        fmt.Printf("pointer to boolean %t\n", *t) // t 是 *bool 类型
    case *int:
        fmt.Printf("pointer to integer %d\n", *t) // t 是 *int 类型
    }
    ```

# 函数

1. Go 与众不同的特性之一就是函数和方法可返回多个值。
2. Go 函数的返回值或结果 “形参” 可被命名，并作为常规变量使用，就像传入的形参一样。 命名后，一旦该函数开始执行，它们就会被初始化为与其类型相应的零值； 若该函数执行了一条不带实参的 return 语句，则结果形参的当前值将被返回。
3. Go 的 `defer` 语句用于预设一个函数调用（即**推迟执行**函数）， 该函数会在执行 `defer` 的函数返回之前立即执行。典型的例子就是解锁互斥和关闭文件。
4. 被推迟的函数按照后进先出（LIFO）的顺序执行

# 数据

1. Go 提供了两种分配原语，即内建函数 `new` 和 `make`。 它们所做的事情各不相同，所应用的类型也不同。它不会 **初始化** 内存，只会将内存 **置零**。 

2. 返回一个局部变量的地址完全没有问题，这点与 C 不同。该局部变量对应的数据 在函数返回后依然有效。

    ```go
    func NewFile(fd int, name string) *File {
        if fd < 0 {
            return nil
        }
        f := File{fd, name, nil, 0}
        return &f
    }
    ```

3. 内建函数 make(T, args) 的目的不同于 new(T)。它只用于创建 slice、map 和 channel，并返回类型为 T（而非 *T）的一个已初始化 （而非置零）的值。 出现这种用差异的原因在于，这三种类型本质上为引用数据类型，它们在使用前必须初始化。

4. `make` 只适用于 `map`、`切片`和 `channel` 且不返回指针。若要获得明确的指针， 请使用 `new` 分配内存。

5. 数组在 Go 和 C 中的主要区别。在 Go 中:

    数组是值。将一个数组赋予另一个数组会复制其所有元素。
    特别地，若将某个数组传入某个函数，它将接收到该数组的一份副本而非指针。
    数组的大小是其类型的一部分。类型 [10]int 和 [20]int 是不同的。

6. 只要切片不超出底层数组的限制，它的长度就是可变的，只需将它赋予其自身的切片即可。

7. 由于切片长度是可变的，因此其内部可能拥有多个不同长度的切片。

8. 赋值和获取映射值的语法类似于数组，不同的是映射的索引不必为整数。

9. 若试图通过映射中不存在的键来取值，就会返回与该映射中项的类型对应的零值。

10. 有时你需要区分某项是不存在还是其值为零值。可以使用多重赋值的形式来分辨这种情况。

    ```go
    var seconds int
    var ok bool
    seconds, ok = timeZone[tz]
    ```

11. 要删除映射中的某项，可使用内建函数 `delete`，它以映射及要被删除的键为实参。 即便对应的键不在该映射中，此操作也是安全的。

12. fmt.Printf 中的 %v 可以打印任意值，%#v 将完全按照 go 的语法打印值。%T 可以打印出某个值的类型

13. 请勿通过调用 Sprintf 来构造 String 方法，因为它会无限递归你的 String 方法

    ```go
    type MyString string
    
    func (m MyString) String() string {
        return fmt.Sprintf("MyString=%s", m) // 错误：会无限递归
    }
    ```

# 初始化

1. 常量只能是数字、字符（符文）、字符串或布尔值。

2. 在 Go 中，枚举常量使用枚举器 `iota` 创建。

    ```go
    type ByteSize float64
    
    const (
        _           = iota // 通过赋予空白标识符来忽略第一个值
        KB ByteSize = 1 << (10 * iota)
        MB
        GB
        TB
        PB
        EB
        ZB
        YB
    )
    ```

3. 每个源文件都可以通过定义自己的无参数 `init` 函数来设置一些必要的状态。而它的结束就意味着初始化结束.

4. `init` 函数还常被用在程序真正开始执行前，检验或校正程序的状态。

# 方法

1. 以指针或值为接收者的区别在于：值方法可通过指针和值调用， 而指针方法只能通过指针来调用。

# 接口和其他类型

1. Go 中的接口为指定对象的行为提供了一种方法：如果某样东西可以完成**这个**， 那么它就可以用在**这里**。

2. 类型断言接受一个接口值， 并从中提取指定的明确类型的值。但若它所转换的值中不包含字符串，该程序就会以运行时错误崩溃。为避免这种情况， 需使用 “逗号，ok” 惯用测试它能安全地判断该值是否为字符串：

    ```go
    str, ok := value.(string)
    if ok {
        fmt.Printf("string value is: %q\n", str)
    } else {
        fmt.Printf("value is not a string\n")
    }
    ```

# 空白标识符

1.  空白标识符可被赋予或声明为任何类型的任何值，而其值会被无害地丢弃
2.  一个类型无需显式地声明它实现了某个接口。取而代之，该类型只要实现了某个接口的方法， 其实就实现了该接口。

# 并发

1. 不要通过共享内存来通信，而应通过通信来共享内存

2. Go 协程具有简单的模型：它是与其它 Go 协程并发运行在同一地址空间的函数。

3. 在函数或方法前添加 `go` 关键字能够在新的 Go 协程中调用它。当调用完成后， 该 Go 协程也会安静地退出。

4. 在 Go 中，匿名函数都是闭包：其实现在保证了函数内引用变量的生命周期与函数的活动时间相同。

5. 若信道是不带缓冲的，那么在接收者收到值前， 发送者会一直阻塞；若信道是带缓冲的，则发送者仅在值被复制到缓冲区前阻塞； 若缓冲区已满，发送者会一直等待直到某个接收者取出一个值为止。

6. 带缓冲的信道可被用作信号量，例如限制吞吐量。

7. Go 协程中的闭包

    ```go
    func Serve(queue chan *Request) {
        for req := range queue {
            sem <- 1
            go func() {
                process(req) // Buggy; see explanation below.
                <-sem
            }()
        }
    }
    ```

    Bug 出现在 Go 的 `for` 循环中，该循环变量在每次迭代时会被重用，因此 `req` 变量会在所有的 Go 协程间共享，这不是我们想要的。我们需要确保 `req` 对于每个 Go 协程来说都是唯一的。

    ```go
    func Serve(queue chan *Request) {
        for req := range queue {
            sem <- 1
            go func(req *Request) {
                process(req)
                <-sem
            }(req)
        }
    }
    ```

8. 并发是用可独立执行组件构造程序的方法， 而并行则是为了效率在多 CPU 上平行地进行计算。尽管 Go 的并发特性能够让某些问题更易构造成并行计算， 但 Go 仍然是种并发而非并行的语言，且 Go 的模型并不适合所有的并行问题

# 错误

1. 真正的库函数应避免 `panic`。若问题可以被屏蔽或解决， 最好就是让程序继续运行而不是终止整个程序。一个可能的反例就是初始化：若某个库真的不能让自己工作，且有足够理由产生 Panic，那就由它去吧。

2. 当 panic 被调用后（包括不明确的运行时错误，例如切片越界访问或类型断言失败）， 程序将立刻终止当前函数的执行，并开始回溯 Go 协程的栈，运行任何被推迟的函数。 若回溯到达 Go 协程栈的顶端，程序就会终止。不过我们可以用内建的 recover 函数来重新或来取回 Go 协程的控制权限并使其恢复正常执行。

3. `recover` 只能在被推迟的函数中才有效。

4. `recover` 的一个应用就是在服务器中终止失败的 Go 协程而无需杀死其它正在执行的 Go 协程。

    ```go
    func server(workChan <-chan *Work) {
        for work := range workChan {
            go safelyDo(work)
        }
    }
    
    func safelyDo(work *Work) {
        defer func() {
            if err := recover(); err != nil {
                log.Println("work failed:", err)
            }
        }()
        do(work)
    }
    ```



参考链接

https://learnku.com/docs/effective-go/