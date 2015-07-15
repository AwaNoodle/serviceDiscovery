# -*- mode: ruby -*-
# vi: set ft=ruby :

$updateDocker=<<END

END

$addServerContainers=<<END
  docker pull gliderlabs/consul-server
END

$addAgentContainers=<<END
  docker pull gliderlabs/consul-agent
  docker pull gliderlabs/registrator:latest
  docker pull yungsang/busybox
END

Vagrant.configure(2) do |config|
  config.vm.box = "yungsang/boot2docker"

  config.vm.define "server" do |server|
    server.vm.network "private_network", ip: "192.168.10.1"
  end

  config.vm.define "agent" do |agent|
    agent.vm.network "private_network", ip: "192.168.10.2"
  end

  # config.vm.synced_folder ".", "/vagrant" 

  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.network "public_network"
  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provision "Update Docker", type: "shell", inline: $updateDocker
end
