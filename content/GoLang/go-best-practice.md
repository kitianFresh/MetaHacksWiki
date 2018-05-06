---
title: "go-best-practice"
date: 2018-04-21 10:54
---

[TOC]

# Go 项目测试（Go Testing）
Go语言这种静态类型的语言，不想Python直接使用PDB很容易调试程序，项目最好是一个功能写完就测试其正确性，这样才能保证最后整个项目的正确性。特别是新手，如果不好好写测试，程序可能跑起来都不知道自己哪里错了。Go 语言测试也是很方便的。

## 单元测试
Go的测试文件都是以 `xxx_test.go`命名的，一般和一个模块或者文件名一一对应，你的测试函数写在这些文件之后即可。测试函数分以Test为前缀的普通测试函数，以Benchmark为前缀的性能测试函数（会被Go Testing调用多次），以Example为前缀的文档实例函数。 Go 项目测试命令是 `go test`, 如果什么都不跟，默认和 `go build` 命令一样会采用当前目录对应的包进行测试。
```go
import "testing"
func TestSin(t *testing.T) { /* ... */ }
func TestCos(t *testing.T) { /* ... */ }
func TestLog(t *testing.T) { /* ... */ }
```
Go 语言惯用的方式是表驱动测试。
```go
func TestIsPalindrome(t *testing.T) {
    var tests = []struct {
        input string
        want  bool
    }{
        {"", true},
        {"a", true},
        {"aa", true},
        {"ab", false},
        {"kayak", true},
        {"detartrated", true},
        {"A man, a plan, a canal: Panama", true},
        {"Evil I did dwell; lewd did I live.", true},
        {"Able was I ere I saw Elba", true},
        {"été", true},
        {"Et se resservir, ivresse reste.", true},
        {"palindrome", false}, // non-palindrome
        {"desserts", false},   // semi-palindrome
    }
    for _, test := range tests {
        if got := IsPalindrome(test.input); got != test.want {
            t.Errorf("IsPalindrome(%q) = %v", test.input, got)
        }
    }
}
```
## 测试单个文件或者单个函数
测试整个包，直接到包目录运行`go test`, 或者运行 `go test cloud-go/pkg/glance-go/store`。

如果你不想测试整个包，而是单个文件，这个时候你得注意了！如果你直接用 `go test xxx_test.go`, 可能会一直报 `yyy variable not defined`. 你明明已经定义了，其实这和 gcc 编译器一样的，如果你 test 文件里边引用了其他文件或者包的变量，你必须在后面添加相应的文件，指明依赖。因此这个比较蛋疼。所以我们还可以采用正则匹配函数名的形式，单独测试某些函数即可。

采用命令 `go test -v -run="French|Canal"` 会自动执行名字含有 `French` 或者 `Canal` 的测试函数。里面是正则匹配，可以写`^French$`只匹配一个。 但是执行函数测试，你必须到函数所在的包目录下执行才行。

## 黑盒测试和白盒测试
黑盒测试只关注某个函数的功能输出是否和预期相同，不关心包内部变量的变化，如果需要查看包内数据结构和变量的变化，得采用白盒测试。

什么时候需要白盒测试，比如下面的例子，有的时候某些功能是需要和数据库或者其他的网络文件打交道的，这些函数我们想要在测试的时候直接替换成其他的我们写的函数，那么测试框架必须要知道替换谁，能够访问到包的内部函数以及变量。这里需要注意的时，每一次测试完成之后一定要修改回去原来的值，以避免影响其他测试正常进行。

如下的代码就测试函数的时候，就修改了该函数会用到的另一个和网络服务打交道的函数 `notifyUser`, 我们对其进行了修改，很好的是，Go中函数是First-Class对象，可以直接修改和保存。
```go
func TestCheckQuotaNotifiesUser(t *testing.T) {
    // Save and restore original notifyUser.
    saved := notifyUser
    defer func() { notifyUser = saved }()

    // Install the test's fake notifyUser.
    var notifiedUser, notifiedMsg string
    notifyUser = func(user, msg string) {
        notifiedUser, notifiedMsg = user, msg
    }
    // ...rest of test...
}
```

## 扩展测试
解决循环依赖问题和集成测试多个组件的时候包导入循环依赖，Go语言是不支持循环依赖的。见《GO语言设计》扩展测试部分。一般是通过在要测试的包中建立一个test包，不把测试文件写在改包中，而是写在另一个test包中。

## Go测试未提供的东西
GO语言没有提供比如Java和Python中通过设置一些“setup”和“teardown”的钩子函数来执行测试用例运行的初始化和之后的清理操作，以及很多类似assert断言，值比较函数，格式化输出错误信息和停止一个识别的测试等辅助函数（通常使用异常机制）。

## Http 框架测试
 - [Testing Your (HTTP) Handlers in Go](https://blog.questionable.services/article/testing-http-handlers-go/)

## Gin 框架的测试
 - [Test-driven Development of Go Web Applications with Gin](https://semaphoreci.com/community/tutorials/test-driven-development-of-go-web-applications-with-gin)
```go
import (
   "encoding/json"
   "net/http"
   "net/http/httptest"
   "testing"
   "github.com/gin-gonic/gin"
   "github.com/stretchr/testify/assert"
)
func performRequest(r http.Handler, method, path string) *httptest.ResponseRecorder {
   req, _ := http.NewRequest(method, path, nil)
   w := httptest.NewRecorder()
   r.ServeHTTP(w, req)
   return w
}
func TestHelloWorld(t *testing.T) {
   // Build our expected body
   body := gin.H{
      "hello": "world",
   }
   // Grab our router
   router := SetupRouter()
   // Perform a GET request with that handler.
   w := performRequest(router, "GET", "/")
   // Assert we encoded correctly,
   // the request gives a 200
   assert.Equal(t, http.StatusOK, w.Code)
   // Convert the JSON response to a map
   var response map[string]string
   err := json.Unmarshal([]byte(w.Body.String()), &response)
   // Grab the value & whether or not it exists
   value, exists := response["hello"]
   // Make some assertions on the correctness of the response.
   assert.Nil(t, err)
   assert.True(t, exists)
   assert.Equal(t, body["hello"], value)
}
```

## 测试参数
1. 测试的时候无法输出glog日志到控制台
`go test -logtostderr=true  -v -run 'TestSwiftClientLargeObject'`
2. 测试的时候因为IO等待10分钟以上被导致进程被kill
`go test -timeout 30m  -v -run ^TestSwiftClientLargeObject$`


# Go 项目依赖管理
## Go 项目目录
一般GOPATH设置的是所有GO项目的根目录，比如`GOPATH="/home/user/go-projects:/usr/local/go"`, 比如我们的项目一般在`go-projects`目录下，该目录组织结构是这样的，没有就新建文件夹。
```python
.
├── bin
├── pkg
└── src
```
那我们的项目在哪里，源码都会放在src目录下，项目的打包都在pkg目录，二进制文件都在bin目录，我们自己的项目和第三方库都是在src目录下面。比如我的几个不同的项目 `cloud-go`, `client-go`, 第三方库项目都会在`github.com`目录下面。
```
├── bin
│   ├── fillstruct
│   ├── go-find-references
│   ├── go-outline
├── pkg
│   └── darwin_amd64
└── src
    ├── client-go
    ├── cloud-go
    ├── github.com
    ├── golang.org
    ├── paas-dbha
    └── sourcegraph.com
```
## Go Get
`go get` 命令自动会找到GOPATH 里的第一个目录，让后把第三方库下载安装到上面的src目录里。
## Go Vendor
自己项目的包需要单独管理，而不是公用公共的包的时候，我们可以使用 `govendor` 工具。在自己项目根目录下面执行`govendor init`,即会生产一个 `vendor`目录。

如果你需要在自己的项目中引用所有需要的库，可以在你自己的项目目录下执行 `govendor add +external`，他会将GOPATH目录下下载的包，拷贝到你自己的项目的vendor目录下面。 `vendor.json` 文件里面记录了所有引用的包。如果你的外部依赖更新了，但是vendor里面没有重新加入，有时候会出现模型奇妙的参数类型错误。
```
├── Makefile
├── README.md
├── cmd
│   ├── glance-go
├── deploy
│   ├── glance-go
├── etc
│   ├── glance-go.toml
├── pkg
│   ├── glance-go
│   ├── util
│   └── version
└── vendor
    ├── git.openstack.org
    ├── git.sankuai.com
    ├── github.com
    ├── golang.org
    ├── gopkg.in
    ├── k8s.io
    └── vendor.json
```
### govendor更多命令
 - [govendor](https://github.com/kardianos/govendor)
状态	缩写状态	含义
+local	l	本地包，即项目自身的包组织
+external	e	外部包，即被 $GOPATH 管理，但不在 vendor 目录下
+vendor	v	已被 govendor 管理，即在 vendor 目录下
+std	s	标准库中的包
+unused	u	未使用的包，即包在 vendor 目录下，但项目并没有用到
+missing	m	代码引用了依赖包，但该包并没有找到
+program	p	主程序包，意味着可以编译为执行文件
+outside	 	外部包和缺失的包
+all	 	所有的包
常见的命令如下，格式为 govendor COMMAND。

通过指定包类型，可以过滤仅对指定包进行操作。

命令	功能
init	初始化 vendor 目录
list	列出所有的依赖包
add	添加包到 vendor 目录，如 govendor add +external 添加所有外部包
add PKG_PATH	添加指定的依赖包到 vendor 目录
update	从 $GOPATH 更新依赖包到 vendor 目录
remove	从 vendor 管理中删除依赖
status	列出所有缺失、过期和修改过的包
fetch	添加或更新包到本地 vendor 目录
sync	本地存在 vendor.json 时候拉去依赖包，匹配所记录的版本
get	类似 go get 目录，拉取依赖包到 vendor 目录

### 测试出错
`use of internal package not allowed`, 一般是你引用的包出问题了，在vendor 目录下删除报错的包，然后重新 `govendor add +external`