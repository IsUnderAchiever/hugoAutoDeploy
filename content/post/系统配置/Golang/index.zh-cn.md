---
title: Golang
description: Golang配置
date: 2024-12-21
slug: Golang配置
image: 202412211329268.png
categories:
    - 系统配置
---
# Golang

> 下载[链接](https://golang.google.cn/dl/)
> 选择`windows-amd64.msi`版本即可

配置环境变量

![image-20241005222325284](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211327280.png)

在`Path`下新建如下内容

```
%GOROOT%\bin
```

还需要配置`GOPATH`作为以后存放go项目的位置

![image-20241005221932481](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211327750.png)

通过cmd输入命令查看版本

```sh
go version
```

![image-20241005222018204](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211328112.png)



配置`GoLand`

![image-20241005222734356](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211328325.png)

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, World!")
}
```

![image-20241005223054395](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo03/202412211328495.png)

