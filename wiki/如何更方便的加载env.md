## [如何加载 env](https://github.com/joho/godotenv)

开始运行时, `config/config.go` 需要加载一些配置, 支持从 env 加载, 所以可以在项目目录新建一个.env 文件, 使用 godotenv autoload 自动加载

如果用 S3 就配置 aws s3 相关的, 如果使用 qiniu 来存储，就配置七牛相关的， 其中 QOR_QINIU_ENDPOINT 用你自己创建的融合 CDN 域名，而不是使用 demo 上的 https://up.qiniup.com, 这个 endpoint 是上传文件之后用来获取文件地址的

如果需要用到其他存储，请参考 [qor/oss](https://github.com/qor/oss)

```
import _ "github.com/joho/godotenv/autoload"
```

.env

```
# DEBUG=true

QOR_AWS_ACCESS_KEY_ID=<your-key-id>
QOR_AWS_SECRET_ACCESS_KEY=<your-key>
QOR_AWS_BUCKET=<your-bucket>
QOR_AWS_REGION=<region>

QOR_QINIU_ACCESS_ID=<your-qiniu-access-id>
QOR_QINIU_ACCESS_KEY=<your-qiniu-access-key>
QOR_QINIU_BUCKET=<bucket>
QOR_QINIU_REGION=<region>
QOR_QINIU_ENDPOINT=<cdn>


DBAdapter=postgres
DBName=qor_example
DBPort=5432
DBHost=localhost
DBUser=postgres
DBPassword=postgres

```
