Check Vagrantfile for customizing number of managers and workers inside docker swarm

    vagrant up
    vagrant ssh manager-1
    cd /vagrant
    docker stack deploy -c docker-stack.yml fission

Load url on host:

    http://192.168.50.100

Load traefik dashboard:

    http://192.168.50.100:8080/dashboard/