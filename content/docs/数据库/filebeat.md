ES 索引

```json
PUT /_index_template/laravel-log
{
  "index_patterns": ["laravel-log*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "30-days-default",
      "index.lifecycle.rollover_alias": "laravel-log",
      "number_of_shards": 1,
      "number_of_replicas": 0
    },
    "mappings": {
      "properties": {
        "message": {"type": "text"},
        "context": {"type": "text"},
        "level": {"type": "integer"},
        "level_name": {"type": "keyword"},
        "channel": {"type": "keyword"},
        "datetime": {"type": "date"},
        "extra": {"type": "text"},
        "env": {"type": "keyword"},
        "trace": {"type": "keyword"},
        "method": {"type": "keyword"},
        "url": {"type": "text"},
        "ua": {"type": "text"},
        "referer": {"type": "text"},
        "ip": {"type": "ip"}
      }
    }
  }
}

PUT /laravel-log-test
```

filebeat 配置文件

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
      - /ccsbackend/storage/logs/laravel-*.log
    tags: ["log-bc4"]

setup.template.name: "laravel-log"
setup.template.pattern: "laravel-log*"
setup.template.overwrite: true
setup.template.enabled: false
setup.ilm.enabled: false

output.elasticsearch:
  hosts: ["host.docker.internal:9200"]
  indices:
    - index: "laravel-log-test"
      when.contains:
        tags: "log-bc4"

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

docker 启动

```bash
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="/Users/lynn/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="/Users/lynn/php/ccsbackend:/ccsbackend:ro" \
  elastic/filebeat:7.17.12 filebeat -e --strict.perms=false
```
