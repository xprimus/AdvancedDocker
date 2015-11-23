# AdvancedDocker
## File IO

To run this solution you need to have [Docker!](https://docs.docker.com/v1.8/installation/) installed. If you are unable to install Docker on your machine, there is Vagrant file included to run a ubuntu trusty64 virtual box where you can install docker with this command 
- <code>curl -sSL https://get.docker.com/ | sh</code>

Once you have an environment with docker set up, navigate to the fileio/ folder if you are not already inside the directory. Then follow these commands for the proper execution :
(Note: If using Vagrant and asked for a password, enter <code>vagrant</code> and you may have to use <code>sudo</code> to run your docker commands.)

```
cd output/ 
docker build -t output .              # Build the output socat container
cd ../curl_container
docker build -t curl_container .      # Build the curl container
docker run -d --name output output    # Run the output container in daemon mode
docker run -i -t --link output:output_io curl_container curl output_io:9001    # Run curl command in the curl_container with the option linked to the output container
```

![FileIO demo](http://i.imgur.com/Orjagu7.gif)

## Ambassador Pattern
