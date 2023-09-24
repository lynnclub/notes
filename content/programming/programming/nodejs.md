---
title: "NodeJS问题"
type: "docs"
weight: 9
---

官方网站  
[https://nodejs.org/](https://nodejs.org/)

基础语法  
[https://www.runoob.com/nodejs/nodejs-tutorial.html](https://www.runoob.com/nodejs/nodejs-tutorial.html)

## 版本兼容

nodejs 最新版本会遇到很多版本兼容问题，建议安装次新版本，当前（20230527）建议安装 16.19。

推荐使用nvm管理版本，方便多版本切换。

```shell
brew install nvm
nvm install 16.19
nvm alias default 16.19
```

## 包管理工具

nodejs 依赖包体量非常大，动不动几个G。

1. [npm](https://www.npmjs.com/)：默认包管理工具，体量大。
2. [pnpm](https://pnpm.io/)：高性能包管理工具，共享依赖项，并且不同项目不会重新安装包，从而最小化磁盘使用量。
3. [yarn](https://yarnpkg.com/)：facebook推出，高速和安全的包管理工具。
4. [cnpm](https://npmmirror.com/)：china npm，阿里巴巴为中国用户定制的 npm 镜像。

推荐使用yarn，但是有些项目使用报错，必须用npm。

```shell
npm i -g yarn
```

## 报错记录

错误：unable to resolve dependency tree  
原因：node版本太高导致插件不兼容  
解决：

```shell
npm i --legacy-peer-deps
```

错误：No matching version found for  
原因：找不到指定版本的包  
解决：

如果 [官网](https://www.npmjs.com/) 能查到该版本，需要将代理恢复。

```shell
npm config set registry=http://registry.npmjs.org
```
