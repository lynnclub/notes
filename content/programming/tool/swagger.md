---
title: "Swagger"
type: "docs"
weight: 4
---

## swaggo

[https://blog.csdn.net/qq_41630102/article/details/128411210](https://blog.csdn.net/qq_41630102/article/details/128411210)

安装命令的版本需要与项目依赖版本（github.com/swaggo/swag
）一致，不然可能报错。

```shell
#支持外部包
swag init --parseDependency --parseInternal --parseDepth=1
#封装命令
//go:generate swag init --parseDependency --parseInternal --parseDepth=1
```

## go-swagger

[https://github.com/go-swagger/go-swagger](https://github.com/go-swagger/go-swagger)
