---
title: "NodeJS Issues"
type: "docs"
weight: 9
---

Official Website
[https://nodejs.org/](https://nodejs.org/)

Basic Syntax
[https://www.runoob.com/nodejs/nodejs-tutorial.html](https://www.runoob.com/nodejs/nodejs-tutorial.html)

## Version Compatibility

The latest version of nodejs will encounter many version compatibility issues. It's recommended to install the second-latest version. Currently (20230527), version 16.19 is recommended.

It's recommended to use nvm for version management, making it easy to switch between multiple versions.

```shell
brew install nvm
nvm install 16.19
nvm alias default 16.19
```

## Package Management Tools

NodeJS dependency packages are very large, often several GB.

1. [npm](https://www.npmjs.com/): Default package management tool, large size.
2. [pnpm](https://pnpm.io/): High-performance package management tool, shares dependencies, different projects won't reinstall packages, minimizing disk usage.
3. [yarn](https://yarnpkg.com/): Released by Facebook, fast and secure package management tool.
4. [cnpm](https://npmmirror.com/): China npm, npm mirror customized by Alibaba for Chinese users.

Yarn is recommended, but some projects have errors and must use npm.

```shell
npm i -g yarn
```

## Error Records

Error: unable to resolve dependency tree
Cause: Node version too high causing plugin incompatibility
Solution:

```shell
npm i --legacy-peer-deps
```

Error: No matching version found for
Cause: Cannot find specified version of package
Solution:

If [official website](https://www.npmjs.com/) can find this version, need to restore proxy.

```shell
npm config set registry=http://registry.npmjs.org
```
