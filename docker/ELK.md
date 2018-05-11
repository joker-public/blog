# ELK全流程记录
ELK包括以下技术栈 elasticsearch + logstash(filebeat) + kibana 用于日志收集及分析
并在此基础上形成程序健康判断和程序状态报警

## filebeat
### 配置并使用
filebeat通过配置文件的方式使用，主要的配置文件为filebeat.yml,全部配置项位于filebeat.reference.yml中可用于参考
示例配置如下：
```yml
filebeat.prospectors:
- type: log
  enable: true
  path: /your/path/to/log
```
如果需要收集某个文件夹下所有子文件夹中的日志,配置如下

```yml
filebeat.prospectors:
- type: log
  enable: true
  path: /var/log/*/*.log
```
以上配置会收集/var/log/ 路径下所有子文件夹中的日志

如果收集的日志不经过logstash处理直接扔到elastic的话

```yml
output.elasticsearch:
    hosts: ["192.168.1.127:9200"]
```

如果需要使用内置的kibana面板
```yml
setup.kibana:
    host:"localhost:5601"
```

如果elasticsearch、kibana使用了安全验证，则需要在配置文件设置账号密码

```yml
output.elasticsearch:
    hosts: ["myEshost:9200"]
    username: "elastic"
    password: "elastic"
setup.kibana:
    host: "mykibanahost:5601"
    username: "elastic"
    password: "elastic"
```
运行 
```bash
./filebeat -e -c filebeat.yml -d "publish"
```

## logstash






## logstash-plugin-grok


## logstash-output-elasticsearch 


## elasticsearch

