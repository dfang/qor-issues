# go mod 方式下 i18n 无法加载

在服务器中go run main.go, 打开页面 前端页面i18n无法加载

具体原因参考 config/i18n.go 

config.Root 在 config/config.go

config.Root 写死了

// Root           = os.Getenv("GOPATH") + "/src/github.com/dfang/qor-demo"

改成

Root, _        = os.Getwd()

os.Getwd() 获取的是运行go run main.go 的那个目录
