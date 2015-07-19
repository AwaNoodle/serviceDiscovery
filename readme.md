# serviceDisocvery

A simple test repository for simulating service discovery for Docker containers using [Registrator](https://github.com/gliderlabs/registrator) &&  [Consul](https://github.com/gliderlabs/docker-consul/tree/legacy). Registrator is a handy container that tracks Docker containers starting up and shutting down and reports them to a registrar. 

## Getting Going

The project contains a Vagrantfile to create an environment which will start up all the components to show registration. You will need to have [Vagrant]() and [VirtualBox]() to spin up the environment. Once you have these, simply open a command prompt and move to the project folder. Then

    > vagrant up

This will pull down the base box, the Docker images, and start them all up. Once Vagrant has finished starting up, open up (http://localhost:8500/ui) to display the Consul UI. Here you will see all the registered services. Any container that is exposing a port will be shown with an entry in the services list:

![Consul Services Screen](/images/services.png) 

There are lots of entries as there are lots of ports exposed by the Consul server and one each for the Redis and CAdvisor containers.

## Removing a Service

Registrator will automatically track the service being remove. We can do this by going onto the test environment and shutting down the Redis container:

    > vagrant ssh -c "docker stop some-redis"

The Redis containerr will shut down. If you refresh the services page in the Consul UI you will see that Redis is now gone.

## Adding a Service

Adding the service back in is simply starting or running it:

    > vagrant ssh -c "docker start some-redis"

Refreshing the Consul services page will show Redis again. 

If we start a new service from scratch, we can see how Consul registers the details:

    > vagrant ssh -c "docker run -d -p 6380:6379 --name another-redis -d redis"

There will now be two Redis containers running, just on different ports. This will show in Consul as two entries for Redis as each instance is using the default naming (for registration, not the name given to the container - more on this later). 

![Two Redis instances](/images/two_redis.png)

If you select the Consul [node](http://localhost:8500/ui/#/dc1/nodes/node1) you can see the seperate instances and the ports that they occupy.

![Node Information](/images/node_info.png)

## Changing Tags and Metadata

Registrator allows you to override the data it provides to the registrar. This is helpful for distinguishing services or providing searchable info. You provide this information as part of the run command supplied to Docker:

    > vagrant ssh -c "docker run -d -p 6381:6379 -e "SERVICE_NAME=yet-another-redis" -e "SERVICE_TAGS=redisServer" --name yet-another-redis -d redis"

This will add another service but with the name *yet-another-redis*. If you click on the entry you can see it will also have the tag *redisServer*

![Redis Metadata](/images/metadata.png)

## Getting the Service Details

Consul exposes the service information via DNS or SRV records or via a HTTP API.

### DNS and SRV

Consul acts as a DNS server. Services are named as **<service name>.services.(datacenter.)consul.** (the trailing **.** is important). For example, the services in this example will be under **<service name>.services.dc1.consul.**. You can interrogate Consul's DNS record using **dig**:

    > vagrant ssh -c "dig @127.0.0.1 -p 8600 redis.service.dc1.consul. ANY"

Consul also supports SRV records (which also contain the port information) via **_<service name>._<protocol>.service(.datacenter).consul**:

    > vagrant ssh -c "dig @127.0.0.1 -p 8600 _redis._tcp.service.consul SRV"

You will see the multiple entries for Redis here as the port information is also supplied. 

It's possible to search for alternate protocols (instead of *_tcp*) by using tags. For example, we could run a Docker image using **-e "SERVICE_TAGS=ssh"** and then look up the service using **_someserver._ssh.service.consul**.

### API

Consul exposes a REST API give you information about registered services.

### Listing all available services

    > vagrant ssh -c "curl http://localhost:8500/v1/catalog/services"

### Listing Service Information

    > vagrant ssh -c "curl http://localhost:8500/v1/catalog/service/redis"

This will list all services registered as *redis*. You will get a JSON structure with all address and port information to call the service.