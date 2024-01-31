---
title: "FileBeat"
type: "docs"
weight: 3
---

## ES 索引

POST /\_index_template/standard-log

```json
{
  "index_patterns": ["standard-log-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "30-days-default",
      "index.lifecycle.rollover_alias": "standard-log",
      "number_of_shards": 1,
      "number_of_replicas": 0
    },
    "mappings": {
      "properties": {
        "message": { "type": "text" },
        "context": { "type": "text" },
        "level": { "type": "integer" },
        "level_name": { "type": "keyword" },
        "channel": { "type": "keyword" },
        "datetime": { "type": "date" },
        "extra": { "type": "text" },
        "env": { "type": "keyword" },
        "trace": { "type": "keyword" },
        "method": { "type": "keyword" },
        "url": { "type": "text" },
        "ua": { "type": "text" },
        "referer": { "type": "text" },
        "ip": { "type": "ip" },
        "command": { "type": "text" },
        "memory": { "type": "integer" }
      }
    }
  }
}
```

根据模板创建索引，filebeat 也可以自动创建

POST /standard-log-test

## filebeat 配置文件

```yaml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

filebeat.inputs:
  - type: log
    enabled: true
    json.keys_under_root: true
    json.overwrite_keys: true
    paths:
      - /storage/logs/standard-laravel-*.log
    tags: ["log-laravel"]

setup.template.name: "standard-log"
setup.template.pattern: "standard-log-*"
setup.template.overwrite: true
setup.template.enabled: false
setup.ilm.enabled: true

output.elasticsearch:
  hosts: ["host.docker.internal:9200"]
  indices:
    - index: "standard-log-laravel"
      when.contains:
        tags: "log-laravel"

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
  - drop_fields:
      fields:
        [
          "agent",
          "cloud",
          "host",
          "host.architecture",
          "host.containerized",
          "host.hostname",
          "host.id",
          "host.mac",
          "host.name",
          "host.os.family",
          "host.os.kernel",
          "host.os.name",
          "host.os.platform",
          "host.os.version",
          "host.type",
          "log.offset",
        ]
```

## docker 启动

```shell
docker run -d --restart=always \
  --name=filebeat_test \
  --user=root \
  --volume="/Users/lynn/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/app:/app:ro" \
  elastic/filebeat:7.17.12 filebeat -e --strict.perms=false
```
