## Starting local test environment

### Create Docker node

```bash
docker-machine create -d virtualbox dev
eval "$(docker-machine env dev)"
```

### Start Consul

```bash
docker run -d \
    --name=consul \
    --net=host \
    consul \
    agent -server -bootstrap-expect=1 -ui \
    -client=`docker-machine ip $DOCKER_MACHINE_NAME`\
    -advertise=`docker-machine ip $DOCKER_MACHINE_NAME`
```

To register a service in Consul's web API we can use curl:

```bash
curl -vvv -XPUT \
`docker-machine ip $DOCKER_MACHINE_NAME`:8500/v1/agent/service/register \
-d '{
 "ID": "graylog_server_1",
 "Name":"graylog_server_1",
 "Port": 29000,
 "tags": ["graylog"]
}'
```

Then we can query Consul DNS API for the service using dig:

```bash
dig @`docker-machine ip $DOCKER_MACHINE_NAME` -p 8600 graylog_server_1.service.consul SRV
```

Lets get some more details about graylog service using REST API:
```bash
curl `docker-machine ip $DOCKER_MACHINE_NAME`:8500/v1/catalog/services?pretty
curl `docker-machine ip $DOCKER_MACHINE_NAME`:8500/v1/catalog/service/graylog_server_1?pretty
```

### Start Registrator
```bash
docker run -d \
    --name=registrator \
    --net=host \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
      consul://`docker-machine ip $DOCKER_MACHINE_NAME`:8500
```

### Start Consul Template

Copy haproxy.ctmpl to /tmp directory in docker-machine

```bash
docker run --net host -it -v /tmp:/tmp \
    camilocot/docker-image-consul-template \
    -consul `docker-machine ip $DOCKER_MACHINE_NAME`:8500 \
    -template /tmp/haproxy.ctmpl:/tmp/haproxy.cfg \
    -log-level debug
```

### Start haproxy
```bash
docker run -d --name haproxy \
   -v /tmp/consul.result:/usr/local/etc/haproxy/haproxy.cfg:ro \
   haproxy:1.5
```

### Start elasticseach
```bash
docker run --net host --name elasticsearch -d elasticsearch -Des.cluster.name="graylog"
```

### Start mongodb
```bash
docker run --net host --name mongo -d mongo
```

### @Todo
Reload haproxy after restart
Run consul as DNS provider for all container
Run all graylog components (sharing ports ???)

