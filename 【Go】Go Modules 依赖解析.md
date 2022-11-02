## 包（package）和模块（module）

>"module" != "package"，模块和包的关系更像是集合和元素的关系，包属于模块，一个模块是零个或者多个包的集合。

```

import (
    // Go 标准包
    "fmt"

    // 第三方包
    "github.com/spf13/pflag"

    // 匿名包
     _ "github.com/jinzhu/gorm/dialects/mysql"

     // 内部包
    "github.com/marmotedu/iam/internal/apiserver"
)
```

这里的fmt、github.com/spf13/pflag和github.com/marmotedu/iam/internal/apiserver都是 Go 包。Go 中有 4 种类型的包。
- Go 标准包：在 Go 源码目录下，随 Go 一起发布的包。
- 第三方包：第三方提供的包，比如来自于 github.com 的包。
- 匿名包：只导入而不使用的包。通常情况下，**我们只是想使用导入包产生的副作用，即引用包级别的变量、常量、结构体、接口等，以及执行导入包的init()函数**。
- 内部包：项目内部的包，位于项目目录下。

## Go Modules 概念 `6-2-2-1-1`

- 六个环境变量：GO111MODULE、GOPROXY、GONOPROXY、GOSUMDB、GONOSUMDB、GOPRIVATE。
- 两个概念：Go module proxy 和 Go checksum database。
- 两个主要文件：go.mod 和 go.sum。
- 一个主要管理命令：go mod。
- 一个 build flag。

### 环境变量

- GO111MODULE：1.14以后默认打开go module
- GOPROXY：模块下载代理(常用GOPROXY="https://goproxy.cn,https://goproxy.io,direct" 
	- 多个地址用`,`隔开。如果请求前一个时响应了404或410，那么自动fallback到下一个
	- direct的作用是直接回源（源指的是比如 GitHub 这种代码托管服务）拉取代码
- GOSUMDB： 用于指定检验模块包的数据库，默认值是sum.golang.org。该服务可以用来查询依赖包指定版本的哈希值，保证拉取到的模块版本数据没有经过篡改。
- GONOPROXY：私有模块放在私有开发仓库时，指定该环境变量避免代理
	- 例如 server项目中 hypers-tool模块下载：GONOPROXY="gitlab.hypers.com"
- GONOSUMDB：指定私有模块无需校验
- GOPRIVATE： 指定私有模块的私有开发仓库，GONOPROXY和GONOSUMDB需要与GOPRIVATE变量值保持一致

### Go Modules 命令

- download：下载 go.mod 文件中记录的所有依赖包。
- edit：编辑 go.mod 文件。
- graph：查看现有的依赖结构。
- init：把当前目录初始化为一个新模块。
- tidy：添加丢失的模块，并移除无用的模块。默认情况下，Go 不会移除 go.mod 文件中的无用依赖。当依赖包不再使用了，可以使用go mod tidy命令来清除它。
- vendor：将所有依赖包存到当前目录下的 vendor 目录下。
- verify：检查当前模块的依赖是否已经存储在本地下载的源代码缓存中，以及检查下载后是否有修改。
- why：查看为什么需要依赖某模块。

#### go get 指定版本号下载

```
go get <package[@version]>
```

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202210241523575.png)

##### go get 如何解决模块依赖冲突

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202210241526148.png)
在上述依赖中，模块 A 依赖了模块 B 和模块 C，模块 B 依赖了模块 D，模块 C 依赖了模块 D 和模块 F，模块 D 又依赖了模块 E。并且，同模块的不同版本还依赖了对应模块的不同版本。

Go Modules 会把每个模块的依赖版本清单都整理出来，最终得到一个构建清单，如下图所示：

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202210241527102.png)

上图中，rough list 和 final list 的区别在于重复引用的模块 D（v1.3、v1.4），最终清单选用了 D 的v1.4版本。

这样做的主要原因有两个。第一个是语义化版本的控制。因为模块 D 的v1.3和v1.4版本变更都属于次版本号的变更，而在语义化版本的约束下，v1.4必须要向下兼容v1.3，因此我们要选择高版本的v1.4。

第二个是模块导入路径的规范。主版本号不同，模块的导入路径就不一样。所以，如果出现不兼容的情况，主版本号会改变，例如从 v1 变为 v2，模块的导入路径也就改变了，因此不会影响 v1 版本

### go.mod和go.sum介绍

#### go.mod文件介绍

```

module github.com/marmotedu/iam

go 1.14

require (
  github.com/AlekSi/pointer v1.1.0
  github.com/appleboy/gin-jwt/v2 v2.6.3
  github.com/asaskevich/govalidator v0.0.0-20200428143746-21a406dcc535
  github.com/gin-gonic/gin v1.6.3
  github.com/golangci/golangci-lint v1.30.0 // indirect
  github.com/google/uuid v1.0.0
    github.com/blang/semver v3.5.0+incompatible
    golang.org/x/text v0.3.2
)

replace (
    github.com/gin-gonic/gin => /home/colin/gin
    golang.org/x/text v0.3.2 => github.com/golang/text v0.3.2
)

exclude (
    github.com/google/uuid v1.1.0
)
```

1. go.mod 语句
go.mod 文件中包含了 4 个语句，分别是 module、require、replace 和 exclude。
- module：用来定义当前项目的模块路径。
- go：用来设置预期的 Go 版本，目前只是起标识作用。
- require：用来设置一个特定的模块版本，格式为<导入包路径> <版本> [// indirect]。
- exclude：用来从使用中排除一个特定的模块版本，如果我们知道模块的某个版本有严重的问题，就可以使用 exclude 将该版本排除掉。
- replace：用来将一个模块版本替换为另外一个模块版本。格式为 $module => $newmodule ，$newmodule可以是本地磁盘的相对路径，例如github.com/gin-gonic/gin => ./gin。也可以是本地磁盘的绝对路径，例如github.com/gin-gonic/gin => /home/lk/gin。还可以是网络路径，例如golang.org/x/text v0.3.2 => github.com/golang/text v0.3.2。

这里需要注意，虽然我们用`$newmodule`替换了`$module`，但是在代码中的导入路径仍然为$module。

replace 在实际开发中经常用到，下面的场景可能需要用到 replace：
- 在开启 Go Modules 后，缓存的依赖包是只读的，但在日常开发调试中，我们可能需要修改依赖包的代码来进行调试，这时可以将依赖包另存到一个新的位置，并在 go.mod 中替换这个包。
- 如果一些依赖包在 Go 命令运行时无法下载，就可以通过其他途径下载该依赖包，上传到开发构建机，并在 go.mod 中替换为这个包。
- 在项目开发初期，A 项目依赖 B 项目的包，但 B 项目因为种种原因没有 push 到仓库，这时也可以在 go.mod 中把依赖包替换为 B 项目的本地磁盘路径。
- 在国内访问 golang.org/x 的各个包都需要翻墙，可以在 go.mod 中使用 replace，替换成 GitHub 上对应的库，例如golang.org/x/text v0.3.0 => github.com/golang/text v0.3.0。

有一点要注意，exclude 和 replace 只作用于当前主模块，不影响主模块所依赖的其他模块。

2. go.mod 版本号

- 如果模块具有符合语义化版本格式的 tag，会直接展示 tag 的值，例如 github.com/AlekSi/pointer v1.1.0 。
- 除了 v0 和 v1 外，主版本号必须显试地出现在模块路径的尾部，例如github.com/appleboy/gin-jwt/v2 v2.6.3。
- 对于没有 tag 的模块，Go 命令会选择 master 分支上最新的 commit，并根据 commit 时间和哈希值生成一个符合语义化版本的版本号，例如github.com/asaskevich/govalidator v0.0.0-20200428143746-21a406dcc535。
- 如果模块名字跟版本不符合规范，例如模块的名字为github.com/blang/semver，但是版本为 v3.5.0（正常应该是github.com/blang/semver/v3），go 会在 go.mod 的版本号后加+incompatible表示。
- 如果 go.mod 中的包是间接依赖，则会添加// indirect注释，例如github.com/golangci/golangci-lint v1.30.0 // indirect。

>这里要注意，Go Modules 要求模块的版本号格式为v..，如果版本号大于 1，它的版本号还要体现在模块名字中，例如模块github.com/blang/semver版本号增长到v3.x.x，则模块名应为github.com/blang/semver/v3。

这里再详细介绍下出现// indirect的情况。原则上 go.mod 中出现的都是直接依赖，但是下面的两种情况只要出现一种，就会在 go.mod 中添加间接依赖

- 直接依赖未启用 Go Modules：如果模块 A 依赖模块 B，模块 B 依赖 B1 和 B2，但是 B 没有 go.mod 文件，则 B1 和 B2 会记录到 A 的 go.mod 文件中，并在最后加上// indirect。
- 直接依赖 go.mod 文件中缺失部分依赖：如果模块 A 依赖模块 B，模块 B 依赖 B1 和 B2，B 有 go.mod 文件，但是只有 B1 被记录在 B 的 go.mod 文件中，这时候 B2 会被记录到 A 的 go.mod 文件中，并在最后加上// indirect。

3. go.mod 文件修改方法

go mod 子命令修改

```
go mod edit -fmt  # go.mod 格式化
go mod edit -require=golang.org/x/text@v0.3.3  # 添加一个依赖
go mod edit -droprequire=golang.org/x/text # require的反向操作，移除一个依赖
go mod edit -replace=github.com/gin-gonic/gin=/home/colin/gin # 替换模块版本
go mod edit -dropreplace=github.com/gin-gonic/gin # replace的反向操作
go mod edit -exclude=golang.org/x/text@v0.3.1 # 排除一个特定的模块版本
go mod edit -dropexclude=golang.org/x/text@v0.3.1 # exclude的反向操作
```

#### go.sum 文件介绍

1. go.sum 文件内容
下面是一个 go.sum 文件的内容：

```
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZOaTkIIMiVjBQcw93ERBE4m30iBm00nkL0i8=
golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3Y=
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPXsUe+TKr0=
rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/QiW4=
rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9JXDnKaTXpA=
```

go.sum 文件中，每行记录由模块名、版本、哈希算法和哈希值组成，如 [/go.mod] :。目前，从 Go1.11 到 Go1.14 版本，只有一个算法 SHA-256，用 h1 表示。

正常情况下，每个依赖包会包含两条记录，分别是依赖包所有文件的哈希值和该依赖包 go.mod 的哈希值，例如：

```
rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3Y=
rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPXsUe+TKr0=
```

但是，如果一个依赖包没有 go.mod 文件，就只记录依赖包所有文件的哈希值，也就是只有第一条记录。额外记录 go.mod 的哈希值，主要是为了在计算依赖树时不必下载完整的依赖包版本，只根据 go.mod 即可计算依赖树。

2. go.sum 文件生成
项目中引用一个新的包，执行go get xxx 时会更新go.sum

3. 校验
执行构建时， go命令会从本地缓存中查找所有的依赖包，并计算这些依赖包的哈希值，与go.sum中记录的哈希值比较，如果哈希值不一致，则校验失败，停止构建。

## 总结

模块下载流程

![](https://cdn.jsdelivr.net/gh/Cubcub1/ImageRepo/obsidian/202210241541018.png)

Go modules 的全局缓存。Go modules 中，相同版本的模块只会缓存一份，其他所有模块公用。目前，所有模块版本数据都缓存在 $GOPATH/pkg/mod 和 $GOPATH/pkg/sum 下。
>go clean -modcache 清除所有的缓存。
