# clop
clop 是一款小巧的命令行解析器，麻雀虽小，五脏俱全。(从零实现)

## 状态
可以体验现有功能，第一个版本3月底发布.
## feature
* 支持环境变量绑定
* posix风格命令行支持，支持命令组合，方便实现普通posix 标准命令
* 子命令支持，方便实现git风格命令
* 结构体绑定，没有中间商一样的回调函数

## 内容
- [Installation](#Installation)
- [Quick start](#quick-start)
	- [code](#quick-start-code)
	- [help message](#help-message)
- [1. How to use required tags](#required tags)
- [2. How to implement git style commands](#subcommand)

## Installation
```
go get github.com/guonaihong/clop
```

## Quick start
### quick start code
```go
package main

import (
	"fmt"
	"github.com/guonaihong/clop"
)

type cat struct {
	NumberNonblank bool `clop:"-c;--number-nonblank" 
	                     usage:"number nonempty output lines, overrides"`

	ShowEnds bool `clop:"-E;--show-ends" 
	               usage:"display $ at end of each line"`

	Number bool `clop:"-n;--number" 
	             usage:"number all output lines"`

	SqueezeBlank bool `clop:"-s;--squeeze-blank" 
	                   usage:"suppress repeated empty output lines"`

	ShowTab bool `clop:"-T;--show-tabs" 
	              usage:"display TAB characters as ^I"`

	ShowNonprinting bool `clop:"-v;--show-nonprinting" 
	                      usage:"use ^ and M- notation, except for LFD and TAB" `

	Files []string `clop:"args=files"`
}

func main() {

	c := cat{}
	err := clop.Bind(&c)

	fmt.Printf("%#v, %s\n", c, err)
}

/*
Usage:
    ./cat [Flags] <files> 

Flags:
    -E,--show-ends           display $ at end of each line 
    -T,--show-tabs           display TAB characters as ^I 
    -c,--number-nonblank     number nonempty output lines, overrides 
    -n,--number              number all output lines 
    -s,--squeeze-blank       suppress repeated empty output lines 
    -v,--show-nonprinting    use ^ and M- notation, except for LFD and TAB 

Args:
    <files>
*/
```

### required flag
```go
 package main

import (
    "fmt"
    "github.com/guonaihong/clop"
)

type curl struct {
    Url string `clop:"-u; --url" usage:"url" valid:"required"`
}

func main() {

    c := curl{}
    clop.Bind(&c)
}

```

### subcommand
```go
package main

import (
	"fmt"
	"github.com/guonaihong/clop"
)

type add struct {
	All      bool     `clop:"-A; --all" usage:"add changes from all tracked and untracked files"`
	Force    bool     `clop:"-f; --force" usage:"allow adding otherwise ignored files"`
	Pathspec []string `clop:"args=pathspec"`
}

type mv struct {
	Force bool `clop:"-f; --force" usage:"allow adding otherwise ignored files"`
}

type git struct {
	Add add `clop:"subcommand=add" usage:"Add file contents to the index"`
	Mv  mv  `clop:"subcommand=mv" usage:"Move or rename a file, a directory, or a symlink"`
}

func main() {
	g := git{}
	clop.Bind(&g)
	fmt.Printf("git:%#v\n", g)
	fmt.Printf("git:set mv(%t) or set add(%t)\n", clop.IsSetSubcommand("mv"), clop.IsSetSubcommand("add"))
}
// run:
// ./git add -f

// output:
// git:main.git{Add:main.add{All:false, Force:true, Pathspec:[]string(nil)}, Mv:main.mv{Force:false}}
// git:set mv(false) or set add(true)

```