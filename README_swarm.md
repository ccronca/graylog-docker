# To-Do - Composer not working in multiple nodes in Swarm

> Compose does not use swarm mode to deploy services to multiple nodes in a swarm. All containers will be scheduled on the current node.
More info:
https://github.com/docker/compose/issues/3656

> To deploy your application across the swarm, use the bundle feature of the Docker experimental build.
More info:
https://docs.docker.com/compose/bundles

Note: Some configuration options are not yet supported in the DAB format, including volume mounts


## Starting local swarm environment

```bash
▶ docker-machine create --driver virtualbox manager1
▶ docker-machine create --driver virtualbox worker1
▶ docker-machine create --driver virtualbox worker2
```

### Create a swarm

```bash
▶ eval $(docker-machine env manager1)
▶ docker swarm init --advertise-addr `docker-machine ip manager1`
```

Run ```docker info``` to view the current state of the swarm:

```
▶ docker swarm init --advertise-addr `docker-machine ip manager1`
Swarm initialized: current node (188lady2p7mp546jk6xzv1p0y) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token
SWMTKN-1-5q0fk9v50x8v4xp7f6yf37zfrelr14v1ezruamxwwcn5bwgutn-5n1r4cwuua7758falhecdt593
\
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and
follow the instructions.
```

Run ```docker node ls``` to view information about nodes

### Add nodes to the swarm
#### Add worker1
```bash
▶ eval $(docker-machine env worker1)
▶ docker swarm join \
    --token
SWMTKN-1-5q0fk9v50x8v4xp7f6yf37zfrelr14v1ezruamxwwcn5bwgutn-5n1r4cwuua7758falhecdt593
\
    192.168.99.100:2377

This node joined a swarm as a worker.

```
#### Add worker2
```bash
▶ eval $(docker-machine env worker2)
▶ docker swarm join \
    --token
SWMTKN-1-5q0fk9v50x8v4xp7f6yf37zfrelr14v1ezruamxwwcn5bwgutn-5n1r4cwuua7758falhecdt593
\
    192.168.99.100:2377

This node joined a swarm as a worker.

```
## Create bundle
```bash
▶ DOCKER_MACHINE_IP=`docker-machine ip $DOCKER_MACHINE_NAME` docker-compose bundle  -o graylogdocker
WARNING: Unsupported key 'volumes' in services.haproxy - ignoring
WARNING: Unsupported key 'depends_on' in services.haproxy - ignoring
WARNING: Unsupported key 'volumes' in services.registrator - ignoring
WARNING: Unsupported key 'depends_on' in services.registrator - ignoring
WARNING: Unsupported key 'dns_search' in services.registrator - ignoring
WARNING: Unsupported key 'dns' in services.registrator - ignoring
WARNING: Unsupported key 'dns' in services.mongo - ignoring
WARNING: Unsupported key 'depends_on' in services.mongo - ignoring
WARNING: Unsupported key 'volumes' in services.mongo - ignoring
WARNING: Unsupported key 'dns_search' in services.mongo - ignoring
WARNING: Unsupported key 'volumes' in services.elasticsearch - ignoring
WARNING: Unsupported key 'dns' in services.elasticsearch - ignoring
WARNING: Unsupported key 'depends_on' in services.elasticsearch - ignoring
WARNING: Unsupported key 'dns_search' in services.elasticsearch - ignoring
WARNING: Unsupported key 'volumes' in services.consul-template - ignoring
WARNING: Unsupported key 'depends_on' in services.consul-template - ignoring
WARNING: Unsupported key 'dns_search' in services.consul-template - ignoring
WARNING: Unsupported key 'dns' in services.consul-template - ignoring
WARNING: Unsupported key 'depends_on' in services.graylog-server - ignoring
WARNING: Unsupported key 'volumes' in services.graylog-server - ignoring
Wrote bundle to graylogdocker
```

## Deploy application across swarm
```bash
▶ DOCKER_MACHINE_IP=`docker-machine ip $DOCKER_MACHINE_NAME` docker deploy
graylogdocker
Loading bundle from graylogdocker.dab
Updating service graylogdocker_consul (id: 51c50cipzy12izbns2m6j45zn)
Updating service graylogdocker_consul-template (id: a1ytur3o6i2k7oovg4fwa57gr)
Updating service graylogdocker_elasticsearch (id: 7b8shqfedmavrupzpm776d1q0)
Updating service graylogdocker_graylog-server (id: 78iq6g9o5h72dh65jdg4bp6p7)
Updating service graylogdocker_haproxy (id: 2cgyaitrkza89jfebkdh1ziql)
Updating service graylogdocker_mongo (id: 7co7j8aljp6g8s6bnjgjthtpa)
Updating service graylogdocker_registrator (id: 9ijwpr5zfwv9ng0g897ptft5c)
```



