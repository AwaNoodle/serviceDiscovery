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

