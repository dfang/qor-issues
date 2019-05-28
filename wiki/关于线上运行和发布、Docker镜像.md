# 关于线上运行和发布、Docker 镜像

最简单的运行方式是 配置好 `config/database.yml`， 然后在项目根目录下 `go run main.go`
当然如果使用配置好 env, 连 database.yml 也不用配置, 具体配置什么变量，读 `config/config.go`
也可以参考 `如何更方便的加载 env`

线上运行也可以克隆整个库，然后`go run main.go`

还有种方式就是 go build 生成可执行文件 `qor-example`, 然后 ./qor-example, 但是这个文件必须在项目根目录下运行
copy 到别的地方单独运行，即使配置好数据库连接信息，也会报 `failed to find template: index`
因为 模板文件、config/locales/\*.yml 没有编译到 qor-example， 这也是为什么只能在项目根目录下运行

Dockerfile 则要这样写, 注意后面的三个 COPY 指令

```
# -----------------------------------------------------------------------------
# step 2: exec
# FROM phusion/baseimage:0.11
FROM golang:1.12.5

RUN mkdir /go-app
WORKDIR /go-app
COPY --from=build-step /go/bin/qor-example /go-app/qor-example
COPY --from=build-step /go/bin/seeds /go-app/seeds
COPY app ./app
COPY public ./public
COPY config ./config

CMD ["/go-app/qor-example"]
```

那有什么办法只需一个执行文件 配置好 env 或者 config/database.yml 在项目目录之外也可以运行呢？
而且单文件运行有什么好处呢?

首先 qor-example + 目录 运行， 模板文件、静态资源文件是从文件系统里读取的

单文件运行，资源文件、静态文件统统打包到 `config/bindatafs/templates_bindatafs.go` 里, 然后编译到可执行文件里了

也就是说单文件运行，模板等是从内存中加载的

也许运行更快，但需要测试

但是坏处就是需要改造代码，而且`config/bindatafs/templates_bindatafs.go`文件会很大

改造的思路是

1. 首先要把 app/views， app/home/views 等目录里的静态文件编译到 templates_bindatafs.go

部分代码

```
bindatafs.AssetFS.NameSpace("home").RegisterPath("app/home/views")
bindatafs.AssetFS.NameSpace("orders").RegisterPath("app/orders/views")
bindatafs.AssetFS.NameSpace("account").RegisterPath("app/account/views")
bindatafs.AssetFS.NameSpace("pages").RegisterPath("app/pages/views")
bindatafs.AssetFS.NameSpace("products").RegisterPath("app/products/views")
bindatafs.AssetFS.NameSpace("views").RegisterPath("app/views")
```

2. `app/home/home.go` 等文件里的 `ConfigureApplication` 需要配置 render 从 bindatafs 读取资源

部分代码

```
controller := &Controller{ View: render.New( &render.Config{ AssetFileSystem: bindatafs.AssetFS.NameSpace("home"), }),
```

3. build 的时候带上 tags, 因为 config/ 下的 templates 里加了 +build bindatafs

```
GOOS=darwin GOARCH=amd64 go build -tags bindatafs
```

总之，保证单文件在项目目录之外能够运行，不报`failed to find template` 和 i18n 未正常显示的问题， 就表示改造成功
