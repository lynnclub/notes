---
title: "jmeter"
type: "docs"
weight: 4
---

[https://jmeter.apache.org/](https://jmeter.apache.org/)

## 下载使用

```shell
#Mac启动GUI模式
./bin/jmeter
./bin/jmeter.sh
#分配资源，初始堆内存512MB，最大堆内存2GB
JVM_ARGS="-Xms512m -Xmx2g"
```

Don't use GUI mode for load testing !, only for Test creation and Test debugging.

## 命令行

使用 JMeter 命令行方式发送 POST 请求并包含 JSON 数据，你需要通过 `-J` 参数指定请求参数，并使用 `-t` 参数指定测试计划文件。以下是一个示例命令：

```bash
jmeter -n -t /path/to/your/testplan.jmx -l /path/to/results.jtl -Jcontent_type=application/json
```

解释一下上述命令：

- `-n`: 表示以非 GUI（无界面）模式运行 JMeter。
- `-t /path/to/your/testplan.jmx`: 指定测试计划文件的路径。
- `-l /path/to/results.jtl`: 指定测试结果输出文件的路径。
- `-Jcontent_type=application/json`: 指定请求的 Content-Type 为 application/json。
- `-Jrequest_body='{"key": "value"}'`: 指定请求的 JSON 数据。

你需要根据实际情况修改这些参数，确保它们符合你的测试计划和接口要求。

如果你的测试计划中包含了 HTTP 请求，那么你在测试计划中要设置好请求的路径、方法等信息。通过 `-J` 参数，你可以动态地设置一些参数，比如请求的头信息、请求体内容等。

请注意，JSON 数据中的双引号可能需要适当地转义，以确保命令行解析正确。在上述例子中，JSON 数据已经被单引号括起来，这有助于避免与命令行解析发生冲突。

确保测试计划文件（`.jmx` 文件）中已经正确配置了你的 HTTP 请求，包括服务器地址、路径、方法等信息。
