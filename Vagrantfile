# -*- mode: ruby -*-
# vi: set ft=ruby :

# number of managers; the first one will be leader
NUM_OF_MANAGERS=1
MEMORY_MANAGER=2048
NUM_CPUS_MANAGER=2

# number of workers
NUM_OF_WORKERS=1
MEMORY_WORKER=2048
NUM_CPUS_WORKER=2

# update the linux machines (sudo apt-get update && sudo apt-get autoremove)
UPDATE_MACHINES=false

# if we want to initialize docker swarm
SWARM_INIT_MANAGERS=true
SWARM_INIT_WORKERS=true

# -- Initialize a manager
@initManager = <<SHELL
echo initManager: $*
if [ ! -d "/vagrant/swarm-info" ]; then
  mkdir /vagrant/swarm-info
  chmod 777 /vagrant/swarm-info

  swarm_manager_ip=$2

  echo $swarm_manager_ip > /vagrant/swarm-info/swarm-manager-ip

  docker swarm init --advertise-addr $swarm_manager_ip:2377
  docker swarm join-token -q manager > /vagrant/swarm-info/swarm-token-manager
  docker swarm join-token -q worker > /vagrant/swarm-info/swarm-token-worker
else
  docker swarm join \
    --token `cat /vagrant/swarm-info/swarm-token-manager` \
    `cat /vagrant/swarm-info/swarm-manager-ip`:2377
fi
SHELL

# -- Initialize a swarm worker
@initWorker = <<SHELL
echo initWorker: $*
docker swarm join \
  --token `cat  /vagrant/swarm-info/swarm-token-worker` \
  `cat /vagrant/swarm-info/swarm-manager-ip`:2377
SHELL

Vagrant.configure("2") do |config|
  # -- General configurations
  config.vm.box = "ubuntu/trusty64"
  config.vm.provision "docker"

  config.vm.provider 'virtualbox' do |v|
    v.linked_clone = true if Vagrant::VERSION =~ /^1.8/
  end

  (1..NUM_OF_MANAGERS).each do |mgrNumber|

    _MACHINE_NAME = "manager-#{mgrNumber}"

    config.vm.define _MACHINE_NAME do |node|

      node.vm.hostname = _MACHINE_NAME

      node.vm.network "private_network", ip: "192.168.50.#{99+mgrNumber}"

      node.vm.provider "virtualbox" do |v|
        v.memory = MEMORY_MANAGER
        v.cpus = NUM_CPUS_MANAGER

        v.name = "manager-#{mgrNumber}"

        # This setting makes the NAT engine use the host's resolver mechanisms to handle DNS requests. See <https://www.virtualbox.org/manual/ch09.html#nat-adv-dns>
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        # This setting makes the NAT engine proxy all guest DNS requests to the host's DNS servers. See <https://www.virtualbox.org/manual/ch09.html#nat-adv-dns>
        v.customize ["modifyvm", :id, "--natdnsproxy1", "on"] 
      end

      if (UPDATE_MACHINES)
        node.vm.provision "shell", inline: "sudo apt-get update && sudo apt-get autoremove"
      end

      if (SWARM_INIT_MANAGERS)
        node.vm.provision "shell", inline: @initManager, args: [ "#{NUM_OF_MANAGERS}" , "192.168.50.#{99+mgrNumber}", "#{mgrNumber}" ]
      end

    end
  end

  (1..NUM_OF_WORKERS).each do |workerNumber|

    _WORKER_NAME = "worker-#{workerNumber}"

    config.vm.define _WORKER_NAME do |node|

      node.vm.hostname = _WORKER_NAME

      node.vm.network "private_network", ip: "192.168.50.#{149+workerNumber}"

      node.vm.provider "virtualbox" do |v|
        v.memory = MEMORY_WORKER
        v.cpus = NUM_CPUS_WORKER

        v.name = "worker-#{workerNumber}"

        # This setting makes the NAT engine use the host's resolver mechanisms to handle DNS requests. See <https://www.virtualbox.org/manual/ch09.html#nat-adv-dns>
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        # This setting makes the NAT engine proxy all guest DNS requests to the host's DNS servers. See <https://www.virtualbox.org/manual/ch09.html#nat-adv-dns>
        v.customize ["modifyvm", :id, "--natdnsproxy1", "on"] 
      end

      if (UPDATE_MACHINES)
        node.vm.provision "shell", inline: "sudo apt-get update && sudo apt-get autoremove"
      end

      if (SWARM_INIT_MANAGERS) && (SWARM_INIT_WORKERS)
        node.vm.provision "shell", inline: @initWorker, args: [ "#{NUM_OF_WORKERS}" , "192.168.50.#{149+workerNumber}", "#{workerNumber}" ]
      end

    end
  end

end