# Ansible Seed

This seed  provides a way to speed up the process of developing Ansible playbooks using Docker containers as hosts.
As a by product, it can also be used to create Docker images using Ansible to perform complex provisioning actions.
These actions could, for example, entail the installation of a middleware software, start it up and executing application specific configuration against its API.
If the final hosts for the playbooks are Virtual Machines or physical hosts, then multiple services can be required to run on the hosts. 
In this case, to test the installation of the services in the hosts, a special Docker based image whith systemd can be used to develop the plays.

### Requirements
- Linux environment running Docker (1.6+)
- Ansible 2.+: the Docker connection plugin is used to avoid having to install SSH in the containers.
- docker-py 1.2.3

# Project Overview
  
  This section describes the configuration of the project and how to use it to develop [Ansible playbooks]( http://docs.ansible.com/ansible/playbooks.html) using Docker containers as target hosts.
  
  
### Development Project Structure

The following image shows the project structure when developing and testing playbooks against Docker containers:

![Process](./docs/img/dev.gif)

The following files are used in the process:

- [run.yml](./provision/run.yml): the main playbook used to launch the automation process.
It contains a list of [hosts](./provision/target_host1.yml) to be provisioned. 
- [target_host_n.yml](./provision/target_host1.yml): the playbook that provisions a set of [roles](./provision/roles) into a specific host container. The provided *target_host_n* files are offered as a template and example. They can be changed to suit the project requirements. See the comments section at the top of the files.
- [initialise_container](./provision/initialise_container.yml): a play to spin up a docker container to be used as a host for provisioning. Called by [target_host_n.yml](./provision/target_host1.yml).
- [terminate_container](./provision/terminate_container.yml): a play to stop, commit and delete a docker container. Called by [target_host_n.yml](./provision/target_host1.yml).
- [roles](./provision/roles): a folder containing the roles can can be provisioned to one or more hosts. This folder can also contain [symbolic links](https://en.wikipedia.org/wiki/Symbolic_link) to roles is a central library that can be reused across projects. An example of such library is [Neus](../neus). 

The result of the process is well tested deployment playbooks that can be run on a number of different topologies and target hosts.


#### Container Lifecycle 

The following image shows the lyfecycle of a container used as a target host for testing playbooks:

![Process](./docs/img/proc.gif)
For each host:

1. Create container using the selected Docker base image (this could normally be an image containing the required Standard Operating Environment)
2. Switch the host to the created container and execute one or more roles on the container.
3. Stop the container
4. Commit the container to a new image
5. Delete the container

The new image can be used to create containers to inspect the state of the installation by runing the bash shell within the container.


### Creating complex docker images

There are cases where creating a Docker image from a Dockefile is difficult as there might be a need to have the container running in order to execute configuration on the target middleware (e.g. use the command line interface to configure a JEE Application Server). 
In these circumstances, this project seed can be used to automate such creation.
As it stands at the moment, the created image will have only one layer committed on top of the base image. This can be changed by instructing Ansible to do intermediate commits of the container as required.

#### Working with containers

A few commands useful to remember when working with containers:

- **run shell in a container**: docker exec -i -t container_name bash
- **create and run a container**: docker run --name container_name -d image_name command
- **list running containers**: docker ps 
- **list all containers**: docker ps -a (includes exited containers)
- **list local images**: docker images
- **delete container**: docker rm -f container_name (use -f if container is running)
- **delete image**: docker rmi image_name

**NOTE**: you need admin privileges to execute the above commands. 

For example, to run an image with mariadb, run the following command:

    docker run --name mariadb -d --privileged -p 3306:3306 gatblau/database:1.0 /usr/bin/mysqld_safe
    
where the "--priviliged" flag is only needed if the image has systemd (see [testing virtual machine hosts](./images/centos7-sysd/readme.md) for more information).

### Provisioning Project Structure

Once the required roles have been created, the [deployer](./provision/deploy.yml) file can be targeted to a specific topology defined in the [inventory](./provision/inventory.txt) file.

The following image shows the project structure when executing the playbook against Virtual Machines using the SSH connection plugin:

![Process](./docs/img/prov.gif)




