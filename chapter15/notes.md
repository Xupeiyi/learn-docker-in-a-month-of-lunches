# Chapter 15 Configuring Docker for secure remote access and CI/CD

The Docker command line just sends instructions to the API running on the Docker Engine.

## 15.1 Endpoint options for the Docker API
The command line and the API use the same channel (Linux sockets, Windows named pipes) for communication.   
Explicitly set up configurations for remote access to the Docker Engine. 

You can use plain unsecured HTTP access as the simplest way, but it's a horrible idea.

Use the `host` argument to connect to a remote machine. For example, `docker --host txp://localhost:2375 container ls`.

When you're working with a remote Docker Engine, any commands you send work in the context of that machine. So if you run a container and mount a volume from the local disk, it's the remote machine's disk that the container sees.