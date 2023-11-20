# Chapter 5 Sharing images with Docker Hub and other registries 

## 5.1 Work with registries, repositories, and image tags
Image reference is an image's unique identifier in a registry.

Four parts of an image reference:
> docker.io/diamol/golang:latest  
format: {domain} / {owner account} / {repo name} : {tag}

Domain's default is docker.io. Tag's default is latest.  
Always tag the image to specify the application version.

## 5.2 Pushing your own images to Docker Hub
One image can have several references.




Registries work with image layers. If some layers are already in the registry, they won't be physically uploaded.