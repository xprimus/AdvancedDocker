# AdvancedDocker
## File IO

To run this solution you need to have [Docker!](https://docs.docker.com/v1.8/installation/) installed. If you are unable to install Docker on your machine, there is Vagrant file included to run a ubuntu trusty64 virtual box where you can install docker with this command 
- <code>curl -sSL https://get.docker.com/ | sh</code>

Once you have an environment with docker set up, navigate to the <code>fileio/</code> folder if you are not already inside the directory. Then follow these commands for the proper execution :
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

The ambassador pattern requires two VMs with docker and docker-compose installed on the VMs. Included inside the <code>ambassador/</code> folder are two folders with Vagrant files that you can use to spin up VMs for this task. Simply navigate to each of the folders and run the following commands to setup the VMs: 
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

## Docker Deploy

Inside the dockerdeploy folder is the code to deploy the simple web App to both a green and blue remote. Since docker isn't friendly on Mac OSX, a Vagrantfile is included to operate as the host machine. If Docker works fine on your machine you can go directly into the <code>code/</code> folder. 

After docker and git are available on your host(or virtual) machine, you may continue.

For setup the following commands were executed:
- First a registry is established with
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
- Next the simple node app was cloned from git clone https://github.com/xprimus/App
- In the same folder where <code>App/</code> is present, we initialize a blue and gree bare bones git repositories with the following commands
```
mkdir blue.git
mkdir green.git
cd blue.git
git init --bare
cd ../green.git
git init --bare
```
- Inside blue.git/hooks/post-receive paste these contents
```
#!/bin/sh

# Stop the container 
sudo docker stop blue_app
# Remove the container       
sudo docker rm blue_app
# Pull the latest container from registry
sudo docker pull localhost:5000/my_app:latest
# Run it as blue deployment  
sudo docker run -p 8080:8080 -d --name blue_app my_app
```

- Inside green.git/hooks/post-receive paste these contents
```
#!/bin/sh

# Stop the container 
sudo docker stop gree_app
# Remove the container       
sudo docker rm green_app
# Pull the latest container from registry
sudo docker pull localhost:5000/my_app:latest
# Run it as blue deployment  
sudo docker run -p 8081:8080 -d --name green_app my_app
```
Make sure both post-receive files have executable access with: <code>chmod 777 post-recieve</code>

- After initializing the repos, we have to setup the remote to those repos from the App folder
```
cd ../App/
git remote add blue ../blue.git
git remote add green ../green.git
```
Then set navigate to .git/hooks/pre-commit inside the App/ folder and paste this code and give it executable access
```
#!/bin/sh

# Build container for current app
sudo docker build -t my_app .
# Remove the old registry
sudo docker rmi localhost:5000/my_app:latest
# Tag the newest build container as the lastest code
sudo docker tag my_app localhost:5000/my_app:latest
# Push the registry to be pulled by a specific remote
sudo docker push localhost:5000/my_app:latest
```
- Now you can commit changes in the app folder and push to 
<code>git push blue master</code>
or
<code>git push green master</code>

and then run <code>sudo docker ps</code> to view your new running containers

With the blue deployment being access on <code>curl localhost:8080</code>
and the green deployment being access on <code>curl localhost:8081</code>

![Docker Deploy](http://i.imgur.com/SzaePUG.gif)
