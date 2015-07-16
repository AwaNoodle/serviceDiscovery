# -*- mode: ruby -*-
# vi: set ft=ruby :

$updateDocker=<<END
  /etc/init.d/docker stop
  curl -L -s -o /usr/local/bin/docker https://get.docker.com/builds/Linux/x86_64/docker-latest && \
    chmod +x /usr/local/bin/docker
  /etc/init.d/docker start
END

$addServerContainers=<<END
  #docker pull gliderlabs/consul-server # <- new version of the consul container but no docs
  docker pull progrium/consul
END

$addAgentContainers=<<END
  #docker pull gliderlabs/consul-agent # <- new version of the consul container but no docs
  docker pull progrium/consul
  docker pull gliderlabs/registrator:latest
  
  # Test box
  docker pull yungsang/busybox
END

$runConsulServer=<<END
  docker run -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp -p 8301:8301 -h node1 progrium/consul -server -bootstrap -ui-dir /ui
END

$runConsulClient=<<END
  docker run -d -p 8301:8301 -p 8400:8400 -p 8500:8500 -p 8600:53/udp --name client -h client progrium/consul -join 192.168.10.10
END

Vagrant.configure(2) do |config|
  #config.vm.box = "yungsang/boot2docker"
  config.vm.box = "williamyeh/ubuntu-trusty64-docker"

  config.vm.define "server" do |server|
    server.vm.network "private_network", ip: "192.168.10.10"
    server.vm.network "forwarded_port", guest: 8500, host: 8500 # Consul HTTP
    #server.vm.network "forwarded_port", guest: 8400, host: 8400 # Consul RPC
    #server.vm.network "forwarded_port", guest: 8600, host: 53 # Consul DNS (UDP)
    #server.vm.network "forwarded_port", guest: 8301, host: 8301
    #server.vm.network "forwarded_port", guest: 8301, host: 8302

    server.vm.provision "Install Images", type: "shell", inline: $addServerContainers
    server.vm.provision "Install Images", type: "shell", inline: $runConsulServer
  end

  config.vm.define "agent" do |agent|
    agent.vm.network "private_network", ip: "192.168.10.11"
    #agent.vm.network "forwarded_port", guest: 8500, host: 8501 # Consul HTTP
    #agent.vm.network "forwarded_port", guest: 8400, host: 8401 # Consul RPC

    agent.vm.provision "Install Images", type: "shell", inline: $addAgentContainers
    agent.vm.provision "Install Images", type: "shell", inline: $runConsulClient
  end

  # config.vm.synced_folder ".", "/vagrant" 

  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "public_network"
  # config.vm.synced_folder "../data", "/vagrant_data"

  # config.vm.provision "Update Docker", type: "shell", inline: $updateDocker
end
