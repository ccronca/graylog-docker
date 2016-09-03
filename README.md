## Starting local test environment

### Create Docker node

```bash
docker-machine create -d virtualbox dev
eval "$(docker-machine env dev)"
```

### Start Consul

```bash

docker run -d \
    --name consul --hostname consul \
    -p 53:53/udp -p 8301:8301/udp -p 8500:8500  \
    consul \
    agent -server -bootstrap-expect=1 -ui \
    -client=0.0.0.0
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
dig @`docker-machine ip $DOCKER_MACHINE_NAME` graylog_server_1.service.consul SRV
```

Lets get some more details about graylog service using REST API:
```bash
curl `docker-machine ip $DOCKER_MACHINE_NAME`:8500/v1/catalog/services?pretty
curl `docker-machine ip $DOCKER_MACHINE_NAME`:8500/v1/catalog/service/graylog_server_1?pretty
```

### Start Registrator
```bash
docker run -d \
    --name registrator \
    --hostname registrator \
    --dns=`docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul` \
    --volume=/var/run/docker.sock:/tmp/docker.sock \
    gliderlabs/registrator:latest \
    -internal \
    consul://consul.service.consul:8500
```

### Start Consul Template

Copy haproxy.ctmpl to /tmp directory in docker-machine

```bash
docker-machine scp haproxy.ctmpl development:/tmp
```

Run consul-template
```bash
docker run -it -v /tmp:/tmp \
    --dns=`docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul` \
    camilocot/docker-image-consul-template \
    -consul consul.service.consul:8500 \
    -template /tmp/haproxy.ctmpl:/tmp/haproxy.cfg \
    -log-level debug
```

### Start haproxy
```bash
docker run -d --name haproxy \
    -v /tmp/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro \
    haproxy:1.5
```

### Start elasticseach
```bash
docker run -d \
    --name elasticsearch \
    --hostname elasticsearch \
    --dns=`docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul` \
    elasticsearch -Des.cluster.name="graylog"
```

### Start mongodb
```bash
docker run -d \
    --name mongo \
    --hostname mongo \
    --dns=`docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul` \
     mongo:3
```

### Start 2 graylog instances
Copy haproxy.ctmpl to /tmp directory in docker-machine

```bash
docker-machine scp graylog.conf development:/tmp
```

Start Graylog instances
```bash
docker run -d \
     --hostname graylog1 \
     --name graylog1 \
     --link mongo:mongo --link elasticsearch:elasticsearch \
     -v /tmp/graylog.conf:/usr/share/graylog/data/config/graylog.conf \
     graylog2/server

docker run -d \
     --hostname graylog2 \
     --name graylog2 \
     --link mongo:mongo --link elasticsearch:elasticsearch \
     -v /tmp/graylog.conf:/usr/share/graylog/data/config/graylog.conf \
     graylog2/server
```

### To-do
#### Done
- [x] Run consul as DNS provider for all container
- [x] Run all graylog components
- [x] Set rest api uri to haproxy

#### Next
- [ ] Reload haproxy after config change
- [ ] Create docker compose file
- [ ] Escalate it using swarm

