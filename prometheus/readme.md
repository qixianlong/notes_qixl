# Prometheus

## 安装
本次总共安装三台机器，ip分别为51，52，53,将51作为prometheus server节点，同时也是集群的master节点，52和53是集群的slave节点。

### 1. prometheus-server
1、解压压缩包
2、修改prometheus.yml配置文件

``` yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["192.168.4.51:9090"]
  #主机节点监控    
  - job_name: "node"
    scrape_interval: 5s
    static_configs:
      - targets: ["192.168.4.51:8083"]
        labels:
          group: 'master'
      - targets: ["192.168.4.52:8083","192.168.4.53:8083"]
        labels:
          group: 'slave'
```
3、启动
``` shell
./prometheus --config.file=prometheus.yml &
```

4、访问页面
http://192.168.4.51:9090/graph

### 2. node-exporter

1、解压
2、启动
讲node-exporter包分发到各个节点
``` shell
./node_exporter --web.listen-address 192.168.4.51:8080
./node_exporter --web.listen-address 192.168.4.52:8080 （主机ip和监听端口）
./node_exporter --web.listen-address 192.168.4.53:8080
```

3. 重新加载配置文件
``` shell
kill -s SIGHUP <PID>
```

4. 优雅地关闭实例

``` shell
kill -s SIGTERM <PID>
```

## PromQL查询语句


node_memory_Active_bytes{job="node",instance="192.168.4.52:8083"}