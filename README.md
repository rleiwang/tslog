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
tslog/tslog	0.1.1        	0.1.1      	Time Series Log Management System
```
* deploy tslog chart
```sh
❯❯❯ helm install --name mytslog --set cockroachdb.Replicas=1,cockroachdb.Storage='10Gi',cockroachdb.Resources.memory='2Gi' tslog/tslog
```
* get minikube ip. In this example, ip address is 192.168.64.12
```sh
❯❯❯ minikube ip
192.168.64.12
```
* add the ip address to /etc/hosts
```sh
192.168.64.12    ui.minikube.com
192.168.64.12    injector.minikube.com
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
type: "agent"

# if empty the default value is mac address of the network interface card
id: demo1

# agent private cert. Agent signs connect request with this cert.
# Server could reject client connection request if cert is invalid
cert: 'f1a996701bf9358dc9b6e5eb4a3dd434f44e3cae914a023debdca98a75038e25bf116c5d41bd56c83e7c73c5bd52a92d567dea84f548c56d992d07f936fdd579'

labels:
  env: dev
  app: demo

# default log to console
#logger:
#  file: "agent.log"
#  maxSize: 10
#  maxBackups: 3
#  maxAge: 2

# required for agent and router
server: "ws://injector.minikube.com/mqtt"
```

## 3. ~~Query log~~ [Deprecated]
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
