# Img2color

## 重构与优化

### 支持 webp

在 img2color.go 中 extractMainColor 函数重新将 webp 后缀图像格式进行 webp 的处理。

```golang
package handler

import (
    ···

    "golang.org/x/image/webp" // webp格式
)

    ···

func extractMainColor(imgURL string) (string, error) {

    ···
    var img image.Image

    contentType := resp.Header.Get("Content-Type")
    switch contentType {
    // imaging不支持webp处理，将其改用webp
    case "image/webp":
        img, err = webp.Decode(resp.Body)
    default:
        img, err = imaging.Decode(resp.Body)
    }

    if err != nil {
        return "", err
    }

    ···

}
```

### 本地运行

修改 `img2color.go` 第一行代码:

```golang
package main
```

执行:

```bash
go run api/img2color.go
```

### 自启动

```bash
/etc/systemd/system/img2color.service
```

```bash
[Unit]
Description=Go Socket Service
After=network.target

[Service]
User={username}
Type=simple
ExecStart=/usr/bin/go run api/img2color.go
WorkingDirectory=/mnt/4.860.ssd/img2color-go
ExecStop=/bin/kill -SIGTERM $MAINPID
TimeoutStopSec=20s
Restart=always
RestartSec=1s
StandardOutput=append:/mnt/4.860.ssd/img2color-go/run.log
StandardError=inherit

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl enable img2color.service
sudo systemctl start img2color.service
sudo systemctl status img2color.service
```

### 修改主题配置

假设将内网的服务映射到 `https://img2color.xxx.com:8888`

```yaml
# 主色调相关配置
mainTone:
  enable: true # true or false 文章是否启用获取图片主色调
  mode: both # cdn/api/both cdn模式为图片url+imageAve参数获取主色调，api模式为请求API获取主色调，both模式会先请求cdn参数，无法获取的情况下将请求API获取，可以在文章内配置main_color: '#3e5658'，使用十六进制颜色，则不会请求both/cdn/api获取主色调，而是直接使用配置的颜色
  # 项目地址：https://github.com/anzhiyu-c/img2color-go
  api: https://https://img2color.xxx.com:8888/api?img= # mode为api时可填写
  cover_change: true # 整篇文章跟随cover修改主色调
```



本项目使用go作为基础，具有较高的性能

支持vercel与服务器部署

## vercel部署

1. 点击项目右上角fork叉子

2. 登录[vercel](https://vercel.com/)

3. 在[vercel](https://vercel.com/)导入项目

4. 部署时添加环境变量

5. 国内访问需绑定自定义域名

## 服务器部署

需要go环境

1. 安装依赖
```bash
go mod tidy
```
2. 运行
```
go run /api/img2color.go
```
此处不赘述守护进程。

## 使用

例如：https://img2color-go.vercel.app/api?img=https://npm.elemecdn.com/anzhiyu-blog@1.1.6/img/post/banner/神里.webp

部署后只需要 域名/api 访问

必填参数img: url

.env文件配置说明


| 配置项                  | 说明                                 |
|-------------------------|--------------------------------------|
| REDIS_ADDRESS           | REDIS地址                            |
| REDIS_PASSWORD          | REDIS密码                            |
| USE_REDIS_CACHE         | bool值，是否启用REDIS                 |
| REDIS_DB                | REDIS数据库名                        |
| USE_MONGODB             | bool值，是否启用mongodb               |
| MONGO_URI               | mongodb地址                          |
| MONGO_DB                | mongodb数据库名                      |
| PORT                    | 端口                                 |
| ALLOWED_REFERERS        | 允许的refer域名，支持通配符，如果有多个地址可以用英文半角,隔开 |
