# AdvancedDocker
## File IO

To run this solution you need to have [Docker!](https://docs.docker.com/v1.8/installation/) installed. If you are unable to install Docker on your machine, there is Vagrant file included to run a ubuntu trusty64 virtual box where you can install docker with this command 
- <code>curl -sSL https://get.docker.com/ | sh</code>

Once you have an environment with docker set up, navigate to the fileio/ folder if you are not already inside the directory. Then follow these commands for the proper execution :
(Note: If using Vagrant and asked for a password, enter <code>vagrant</code> and you may have to use <code>sudo</code> to run your docker commands.)

```
cd output/ 
docker build -t output .              # Build the output socat container
cd ../curl_container                  # Navigate to the curl container
docker build -t curl_container .      # Build the curl container
docker run -d --name output output    # Run the output container in daemon mode
docker run -i -t --link output:output_io curl_container curl output_io:9001    # Run a linked curl command

```

![FileIO demo](http://i.imgur.com/Orjagu7.gif)

## Ambassador Pattern

The ambassador pattern requires two VMs with docker and docker-compose installed on the VMs. Included inside the ambassador/ folder are two folders with Vagrant files that you can use to spin up VMs for this task. Simply navigate to each of the folders and run the following commands to setup the VMs: 
```
vagrant up
vagrant ssh
```
 - host1: redis server hosted in a private IP at 192.168.33.10 and listening on port 6379 with a redis ambassador exposing that port
 - host2: redis client hosted in a private IP at 192.168.33.11 and listening on port 8000 and a redis ambassador connecting to 192.168.33.10:6379 over TCP
 
In host1, run <code>docker-compose up</code> to build and run the redis server container and first ambassador container. You may have to prepend the command statement with sudo.

In host2, run <code>docker-compose up -d</code> to build and run the redis client container and second ambassador container.

Since the IP addresses for the VMs are set in the Vagrantfiles, simply run a curl statment to set and get values in the redis server with a command in this format:
```
curl  192.168.33.11:8000/set/somekey/somevalue
curl  192.168.33.11:8000/get/somekey
```

![Ambassador pattern demo](http://i.imgur.com/DKcn59M.gif)
