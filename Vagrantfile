# -*- mode: ruby -*-
# vi: set ft=ruby :

$updateDocker=<<END
  /etc/init.d/docker stop
  curl -L -s -o /usr/local/bin/docker https://get.docker.com/builds/Linux/x86_64/docker-latest && \
    chmod +x /usr/local/bin/docker
  /etc/init.d/docker start
END

$addServerContainers=<<END
  docker pull progrium/consul
END

$addAgentContainers=<<END
  docker pull gliderlabs/registrator:latest
  
  # Test box
  docker pull redis
END

$runContainers=<<END
  docker run -d --name consulserver -p 8302:8302 -p 8301:8301 -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h node1 progrium/consul -server -bootstrap -ui-dir /ui
  JOIN_IP="$(docker inspect -f '{{.NetworkSettings.IPAddress}}' consulserver)"
  echo Consul Server running at $JOIN_IP
  
  docker run -d -v /var/run/docker.sock:/tmp/docker.sock --name registrator -h $HOSTNAME gliderlabs/registrator consul://$JOIN_IP:8500
  
  docker run -p 6379:6379 --name some-redis -d redis
END

Vagrant.configure(2) do |config|
  config.vm.box = "williamyeh/ubuntu-trusty64-docker"

  config.vm.define "server" do |server|
    server.vm.network "forwarded_port", guest: 8500, host: 8500 # Consul HTTP

    server.vm.provision "Install Images", type: "shell", inline: $addServerContainers
    server.vm.provision "Run Consul", type: "shell", inline: $runContainers
    server.vm.provision "Add Registrator", type: "shell", inline: $addAgentContainers
  end
end
