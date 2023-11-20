# Chapter 4 Packaging applications from source code into Docker Images

## 4.1 Who needs a build server when you have a Dockerfile?
Package the build toolset once and share it: 
- write a Dockerfile that script the deployment of all the tools, and build it into an image
- Use that image in the application Dockerfiles to compile the source code

A multi-stage Dockerfile
```
FROM diamol/base AS build-stage
RUN echo 'Building...' > /build.txt

FROM diamol/base AS test-stage
COPY --from=build-stage /build.txt /build.txt
RUN echo 'Testing...' >> /build.txt

FROM diamol/base
COPY --from=test-stage /build.txt /build.txt
CMD cat /build.txt
```
Three stages: build-stage, test-stage, the final unamed stage. The output will be a single Docker image with the contents of the final stage.  
Can copy files from the previous stage with `--from`

The `RUN` instruction executes a command inside the container during build, and any output from that command is saved in the image layer.

Individual stages are isolated. The output in the final stage only contains what's explicitly copied from earlier stages.

In real life it will be:
1. In the build stage, use a base image with build tools installed. Copy the source code and run the build command.
2. In the test stage, use a base image with test tools installed. Copy the compiled binaries from the build stage and run the tests.
3. In the final stage, use a base image with application runtime installed. Copy the binaries from the build stage that has been tested in the test stage.


## 4.2 App walkthrough: Java source code
```
FROM diamol/maven AS builder

WORKDIR /usr/src/iotd
COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY . .
RUN mvn package

# app
FROM diamol/openjdk

WORKDIR /app
COPY --from=builder /usr/src/iotd/target/iotd-service-0.1.0.jar .

EXPOSE 80
ENTRYPOINT ["java", "-jar", "/app/iotd-service-0.1.0.jar"]
```
`RUN mvn -B dependency:go-offline` is an expensive operation so it has its own step.

## 4.3 App walkthrough: Node.js source code
```
FROM diamol/node AS builder

WORKDIR /src
COPY src/package.json

RUN npm install

# app
FROM diamol/node

EXPOSE 80
CMD ["node", "server.js"]

WORKDIR /app
COPY --from=builder /src/node_modules/ app/node_modules/
COPY src/ .
```
(why can the CMD be written before the source code and dependencies are copied?)


## 4.4 App walkthrough: Go source code
```
FROM diamol/golang AS builder
COPY main.go .

RUN go build -o /server

# app
FROM diamol/base

ENV IMAGE_API_URL="http://iotd/image" \
    ACCESS_API_URL="http://accesslog/access-log"
CMD ["/web/server"]

WORKDIR web
COPY index.html .
COPY --from=builder /server .
RUN chmod +x server
```
Go compiles to native binaries. 
Go applications don't usually fetch dependencies.
By using a minimal base image for the application, we saved almost 750 MB.

## 4.5 Understanding multi-stage Dockerfiles
Why its useful:
1. Standardization. Different users will build and run the app in the same way.
2. Performance. Each stage is cached.
3. Build can be fine-tuned and image is lean. Tools aren't present in the final image. 