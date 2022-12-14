1. 记录声明的注释应该是完整的句子，即使这似乎有点多余。

2. 注释应从所描述的事物的名称开始，并以句号结束

3.  Go 程序沿着整个函数调用链显式地传递 Contexts 。大部分使用上下文的函数都要将其作为第一个参数

4. 不要将上下文作为一个成员添加到结构类型；而是将 ctx 参数添加到该类型的每个方法上。

5. 为避免意外的别名，从另一个包复制 struct 时要小心。例如，bytes.Buffer 类型包含一个 []byte 的 slice。如果复制一个 Buffer，副本中的 slice 可能会对原始数组进行别名操作，从而导致后续方法调用产生令人惊讶的效果。

    通常，如果 T 类型的方法与其指针类型 *T 相关联，请不要复制 T 类型的值。

6. 不要使用 math/rand 来生成密钥，即使是一次性密钥。在没有种子（seed）的情况下，生成器是完全可以被预测的。使用 time.Nanoseconds() 作为种子值，熵只有几位。请使用 crypto/rand 的 Reader 作为替代

7. 当声明一个空 slice 时，倾向于用 `var t []string` ,他会申明一个 nil slice 值。 `t := []string` 声明一个非 nil 但是长度为零的 slice

8. 所有的顶级导出的名称都应该有 doc 注释，重要的未导出类型或函数声明也应如此。

9. 不要将 panic 用于正常的错误处理。使用 error 和多返回值。

10. 错误信息不应大写（除非以专有名词或首字母缩略词开头）或以标点符号结尾

11. 添加新包时，请包含预期用法的示例：可运行的示例，或是演示完整调用链的简单测试。

12. 当你生成 goroutines 时，要清楚它们何时或是否会退出。

    通过阻塞 channel 的发送或接收可能会引起 goroutines 的内存泄漏

13. 不要使用 _ 变量丢弃 error。如果函数返回 error，请检查它以确保函数成功。处理 error，返回 error，或者在真正特殊的情况下使用 panic。

14. 包导入按组进行组织，组与组之间有空行。标准库包始终位于第一组中。

15. 名称中的单词是首字母或首字母缩略词（例如 “URL” 或 “NATO” ）需要具有相同的大小写规则。例如，”URL” 应显示为 “URL” 或 “url” （如 “urlPony” 或 “URLPony” ），而不是 “Url”。

16. Go 的接口要包含在使用方的包里，不应该包含在实现方的包里。实现方只需要返回具体类型（通常是指针或结构体），这样一来可以将新方法添加到实现中，而不需要扩展重构。

17. 如果函数只有几行，则可以使用裸返回。一旦它是一个中等大小的函数，请明确说明您的返回值。

18. 以小写单词开头的句子**不属于**程序包注释的可接受选项。

19. 不要只是为了节省几个字节就将指针作为函数参数传递。这个建议不适用于大型结构体 ，甚至不适用于可能变大的小型结构体。

20. 方法接收者的名称应该反映其身份；通常，其类型的一个或两个字母缩写就足够了（例如用 “c” 或 “cl” 表示 “client” ）。如果你在一个方法中叫将接收器命名为 “c”，那么在其他方法中不要把它命名为 “cl”

21. 方法接收者类型

    - 如果接收者是 map，func 或 chan，则不要使用指针。如果接收者是 slice 并且该方法不重新切片或不重新分配切片，则不要使用指针
    - 如果该方法需要改变接收者的值，则必须用指针。
    - 接收者内含有 sync.Mutex 或者类似的同步域，那就必须指针，以避免拷贝。
    - 接收者是一个大数据结构或者数组，指针会效率更高。 多大才算大？假设它相当于将其包含的所有元素作为参数传递给方法。如果感觉太大，那么对接收者来说也太大了。
    - 函数或方法（同时执行或调用某方法时继续调用相关方法或函数）是否会使接收者发生变化？在调用方法时，值类型会创建接收者的副本，因此外部更新将不会应用于此接收者。如果必须在原始接收者中看到更改效果，则接收者必须是指针。
    - 如果接收者是一个结构、数组、slice，且内部的元素有指针指向一些可变元素，那更倾向于用指针接收，来提供更明确的语义。
    - 若接收者是一个小对象或数组，概念上是一个值类型（比如 time.Time），并且没有可变域和指针域，或者干脆就是 int、string 这种基本类型，适合用值接收者。值接收者可以减少垃圾内存，通过它传递值时，会优先尝试从栈上分配内存，而不是堆上。但保不齐在某些情况下编译器没那么聪明，所以这个在用之前要 profiling 一下。
    - 最后，要是把握不准，就用指针。

22. 同步函数使 goroutines 能够在调用中本地化，从而使其在生命周期中更显得合理，避免泄漏和数据竞争。

23. `Go` 中的变量名称尽可能短为妙，言简意赅，尤其是对于那些处于有限空间中的局部变量更是如此。例如： 用 `c` 而不是 `lineCount`； 用 `i` 而不是 `sliceIndex` 。



参考地址：https://learnku.com/go/wikis/48375