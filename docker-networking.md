- [Docker Netwoking example](#docker-networking-example)
- [Docker Networking](#docker-networking)
- [Legacy Linking](#legacy-linking)


Docker Networking example
====

The following example referenced from linkedin docker course

`$` commands are in host terminal

`$$$` commands are inside docker container


    (terminal 1)
    $ docker network create learning
    (creates a new network)

    (terminal 1)
    $ docker run --rm -ti --net learning --name catserver ubuntu:14.04 bash
    (creates a container on the network: learning)

    (terminal 2)
    $ docker run --rm -ti --net learning --name dogserver ubuntu:14.04 bash
    (creates second container on network:learning)

    (terminal 1)
    $$$ ping dogserver
    (ping should be successful)

    (terminal 2)
    $$$ ping catserver
    (ping should be successful)

    (terminal 2)
    $$$ nc -lp 1234
    (starts listening on port 1234)

    (terminal 1)
    $$$ nc dogserver 1234  (connects to dogserver running on terminal 2)
    meow meow meow meow (this message will be sent to terminal 2)
    (if message is typed in terminal 2, it will be reflected in terminal 1)

    (terminal 3)
    $ docker network create catsonly
    (create second network)
    
    (terminal 3)
    $ docker network connect catsonly catserver
    (add catserver to this network)

    (terminal 3)
    $ docker run --rm -ti --net catsonly --name bobcatserver ubuntu:14.04 bash

    (terminal 3)
    $$$ ping catserver
    (ping should be successful) 

    (terminal 3)
    $$$ ping dogserver
    (not accessible)
    

Docker Networking
====

To inspect network info

    $ docker inspect container_name
    $ docker network ls

When docker is installed, it creates 3 network
    
    - Bridge (default):  $docker run ubuntu 
    - None:  $docker run ubuntu --network=none
    - Host: $docker run ubuntu --network=host

- Bridge Network:
    - All containers receive ip address in range `172.17.0.x` and are accessible to one another.
    - Containers can be accessed outside the host thru port mapping

- None network:
    - Container is not accessible outside the host

- Host network:
    - The port on container is the same port as that of the host. 
    - If a webapp is started at port 5000 then another webapp will have to chose a different port on the host

![docker-networks](docker-networks.png)

To create networks within the docker-host

    file:Dockerfile
    
    docker network create \
        --driver bridge \
        --subnet 182.18.0.0.16
        custom-isolated-network

![user-defined-network](user-defined-network.png)

Legacy Linking
====

- Links all ports but only one way

- Secret environment variables are shared only one way

- Depends on startup order

- Restart only sometimes break the links

Commands

    (terminal 1)
    $ docker run --rm -ti -e SECRET=DockerTutorial --name catserver ubuntu:14.04 bash

    (terminal 2)
    $ docker run --rm -ti --link catserver --name dogserver ubuntu:14.04 bash

    (terminal 1)
    $$$ nc -lp 1234

    (terminal 2)
    $$$ nc catserver 1234
    (dogserver will be able to send/receive msgs from catserver)

    (terminal 2)
    $$$ nc -lp 4321

    (terminal 1)
    $$$ nc dogserver 4321
    catserver does not recognize dogserver because links work in one direction

    (terminal 1)
    $$$ env
    SECRET=DockerTutorial
    TERM=xterm
    ....many other env properties....

    (terminal 2)
    $$$ env
    CATSERVER_ENV_SECRET=DockerTutorial
    ...many other env properties....
    (So, env variables were copied from catserver to dogserver but not the other way around)
