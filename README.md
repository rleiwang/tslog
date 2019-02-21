# tslog
This is a demo project to run Time Series Log Management System locally.

1. setup local kubernetes 
2. start tslog agent
3. query log with graphql p

## 1. Deploy tslog HELM chart
--------------------------

[minikube setup notes](https://github.com/rleiwang/tslog/wiki)

* add helm chart repo
```sh
❯❯❯ helm repo add tslog 'https://raw.githubusercontent.com/rleiwang/helm-repo/master/'
❯❯❯ helm repo update
```
* check repo is added and tslog chart is avaiable
```sh
❯❯❯ helm search tslog
NAME       	CHART VERSION	APP VERSION	DESCRIPTION
tslog/tslog	0.1.0        	0.1.0      	Time Series Log Management System
```
* deploy tslog chart
```sh
❯❯❯ helm install --name mytslog tslog/tslog
```
* get minikube ip 
```sh
❯❯❯ minikube ip
192.168.64.*
```
* add the ip address to /etc/hosts
```sh
192.168.64.*    local.minikube.com
```

## 2. Start tslog agent
--------------------

Start tslog agent on log collector machine

```sh
tslog-agent -f agent_config.yaml
```

### agent_config.yaml

tslog agent sends/receives data to/from injector in MQTT format over websocket

```yaml
server: "ws://local.minikube.com/injector/mqtt"
```

agent_config.yaml would need to specify log file and its format

```yaml
# log line format, TIMESTAMP and LOG are reserved field names
format: &f001
  pattern: '^{TIMESTAMP} {component}: {LOG}'
  # TIMESTAMP format layout in Go style
  layout: 2006-01-02T15:04:05.999999999Z07:00

watcher:
  dir: logs
  file: client.log
  appendix: 1
  format: *f001
  # block, max can't exceed 2^16
  block:
    min: 4096
    max: 65535
```

## 3. Query log
--------------------

* Download and install [GraphQL IDE](https://github.com/prisma/graphql-playground)

setup url on GraphQL IDE
```url
http://local.minikube.com/server/graphql
```

* List loggers
```graphql
query {
  loggers {
    id
    dir
    file
    appx
    format {
      id
      pattern
      layout
    }
  }
}
```

* Search log content
```graphql
query {
  select(
    filter: {
      name: "c0155df08d075bed"
      columns: "all"
      range: { beg: "beg", end: "end" }
      where: "LOG.contain('Job') && component.contain('worker')"
    }
  ) {
    columns
    rows
  }
}
```
![alt text](https://github.com/rleiwang/tslog/raw/master/images/logger.png "list loggers")
![alt text](https://github.com/rleiwang/tslog/raw/master/images/content.png "select log content")
