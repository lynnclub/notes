---
title: "grafana"
type: "docs"
weight: 3
---

[https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/](https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/)

## 安装

```shell
# Debian/Ubuntu
apt install grafana
apt install prometheus
# Mac
brew install grafana
brew install prometheus
```

### Origin not allowed

Origin not allowed 错误可能是由于 Grafana 服务器没有正确配置导致的。

在 Grafana 的配置文件（通常是 grafana.conf）中添加允许的来源（origins）。

```shell
http_origin = <your_origin>
```

其中<your_origin>应替换为您的源地址。如果您不确定您的源地址是什么，您可以使用浏览器中的 document.origin 来查看。

在完成上述更改后，重启 Grafana 服务器以应用更改。

## 使用步骤

适用于 Grafana v10.1.5

1. 添加 connection
2. 添加 data source
3. 创建 dashboard
4. 添加 variables 全量变量，比如日期、类型

## 注意

```sql
SELECT date,
SUM(a_user)/SUM(enter_user) AS AUserRate,
SUM(b_user)/SUM(enter_user) AS BUserRate
FROM my_table
WHERE
(${filed} = 'all' OR filed ${filed})
GROUP BY date LIMIT 100
```
