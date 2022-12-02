1. 工作区样例

    ```go
    bin/
        hello                          # 可执行命令
        outyet                         # 可执行命令
    src/
        github.com/golang/example/
            .git/                      # Git代码库元数据
        hello/
            hello.go               # 命令源码
        outyet/
            main.go                # 命令源码
            main_test.go           # 测试源码
        stringutil/
            reverse.go             # 命令源码
            reverse_test.go        # 测试源码
        golang.org/x/image/
            .git/                      # Git代码库元数据
        bmp/
            reader.go              # 命令源码
            writer.go              # 命令源码
        ... (更多库和资源包省略) ...
    ```

2. GOPATH 一定不能与你的 go 安装路径相同

3. 标准库中的包有给定的短路径，比如 fmt 和 net\http，对于你自己的包，你必须选择一个基本路径，来保证它不会与将来添加到标准库、或其他扩展库中的包冲突。

4. 编写第一个库

    第一步是选择一个包路径，这里使用 github.com/user/stringutil 并创建包目录

    `mkdir $GOPATH/src/github.com/user/stringutil`

    接下来创建一个文件

    ```go
    // 包 stringutil 包含处理字符串的实用函数。
    package stringutil
    
    // Reverse 从左到右按运行方式反向返回传入的字符串。
    func Reverse(s string) string {
        r := []rune(s)
        for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
            r[i], r[j] = r[j], r[i]
        }
        return string(r)
    }
    ```

    使用 go build 编译，这不会产生输出文件。相反，他将编译后的包保存在本地构建缓存中。

    在确认构建了 `stringutil` 包之后，修改原始的 `hello.go`（在 `$GOPATH/src/github.com/user/hello` 里面）以使用它：

    ```go
    package main
    
    import (
        "fmt"
    
        "github.com/user/stringutil"
    )
    
    func main() {
        fmt.Println(stringutil.Reverse("!oG ,olleH"))
    }
    ```

    完成以上步骤后，您的工作空间应该是这样的:

    ```go
    bin/
        hello                 # 可执行命令
    src/
        github.com/user/
            hello/
                hello.go      # 命令源码
            stringutil/
                reverse.go    # 包源码
    ```

5. Go 的约定是包名为导入路径的最后一个元素：作为 "crypto/rot13" 导入的包名应该为 “rot13“，可执行命令必须使用 packet main

6. Go 有一个轻量级的测试框架，由 `go test` 命令和 `testing` 包组成。

    通过创建一个以 _test.go 结尾的文件来编写测试，该文件包含名为 TestXXX 的函数，签名为 func (t *testing.T)。测试框架会运行所有这样的函数；如果该函数调用了一个失败的函数，如 t.Error 或 t.Fail，则认为测试失败。

    通过创建包含以下 Go 代码的文件 `$GOPATH/src/github.com/user/stringutil/reverse_test.go`，向 `stringutil` 包添加一个测试。

    ```go
    package stringutil
    
    import "testing"
    
    func TestReverse(t *testing.T) {
        cases := []struct {
            in, want string
        }{
            {"Hello, world", "dlrow ,olleH"},
            {"Hello, 世界", "界世 ,olleH"},
            {"", ""},
        }
        for _, c := range cases {
            got := Reverse(c.in)
            if got != c.want {
                t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
            }
        }
    }
    ```

    然后使用 `go test` 运行测试：

    ```bash
    go test github.com/user/stringutil
    ok      github.com/user/stringutil 0.165s
    ```

    

参考链接：

https://learnku.com/go/wikis/38174