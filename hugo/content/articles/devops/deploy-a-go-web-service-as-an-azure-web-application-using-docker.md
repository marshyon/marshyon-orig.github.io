---
title: "Deploy a Go Web Service as an Azure Web Application Using Docker"
date: 2020-04-10T12:54:00+01:00
authors: ["Jon Brookes"]
draft: false
---

Azure Web Applications can only support a limited number of languages and whilst Go was supported for a time it is no longer so as an alternative is to serve a custom Docker image with a Go web application within it, hosted on a public registry such as Docker Hub or a private one within Azure itself. The following focuses on the use of a private registry hosted in Azure.

To follow the procedure outlined below the following prerequisites will need to be installed :

* Docker
* azure cli
* Go

Pretty much all that follows is run from a bash shell under Linux, Mac or Ubuntu sub system for windows. The command 'jq' will be used and if it is not already installed it will also be needed :

```
sudo apt install jq
```

With the az command installed, it is also assumed that there is already an Azure account available and that this is authenticated and ready to use with

```
az login
```

The following is the server.go file

```
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/gorilla/mux"
	"gopkg.in/natefinch/lumberjack.v2"
)

func handler(w http.ResponseWriter, r *http.Request) {
	query := r.URL.Query()
	name := query.Get("name")
	if name == "" {
		name = "Guest"
	}
	log.Printf("Received request for %s\n", name)
	w.Write([]byte(fmt.Sprintf("Hello, %s\n", name)))
}

func main() {
	// Create Server and Route Handlers
	r := mux.NewRouter()

	r.HandleFunc("/", handler)

	srv := &http.Server{
		Handler:      r,
		Addr:         ":8080",
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 10 * time.Second,
	}

	// Configure Logging
	LOG_FILE_LOCATION := os.Getenv("LOG_FILE_LOCATION")
	if LOG_FILE_LOCATION != "" {
		log.SetOutput(&lumberjack.Logger{
			Filename:   LOG_FILE_LOCATION,
			MaxSize:    500, // megabytes
			MaxBackups: 3,
			MaxAge:     28,   //days
			Compress:   true, // disabled by default
		})
	}

	// Start Server
	go func() {
		log.Println("Starting Server")
		if err := srv.ListenAndServe(); err != nil {
			log.Fatal(err)
		}
	}()

	// Graceful Shutdown
	waitForShutdown(srv)
}

func waitForShutdown(srv *http.Server) {
	interruptChan := make(chan os.Signal, 1)
	signal.Notify(interruptChan, os.Interrupt, syscall.SIGINT, syscall.SIGTERM)

	// Block until we receive our signal.
	<-interruptChan

	// Create a deadline to wait for.
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*10)
	defer cancel()
	srv.Shutdown(ctx)

	log.Println("Shutting down")
	os.Exit(0)
}
```

The following is the Dockerfile which will be used to build a new docker image based on the Go application above

```
FROM golang:latest as builder

# Add Maintainer Info
LABEL maintainer="Jon Brookes <marshyon@gmail.com>"

# Set the Current Working Directory inside the container
WORKDIR /app

# Copy go mod and sum files
COPY go.mod go.sum ./

# Download all dependencies. Dependencies will be cached if the go.mod and go.sum files are not changed
RUN go mod download

# Copy the source from the current directory to the Working Directory inside the container
COPY . .

# Build the Go app
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main 
FROM alpine:latest  

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy the Pre-built binary file from the previous stage
COPY --from=builder /app/main .

# Expose port 8080 to the outside world
EXPOSE 8080

# Command to run the executable
CMD ["./main"]
```

Here is the directory structure for the above files:

```
go-docker-app/
├── Dockerfile
└── server.go
```

Create the above directory, cd into it, create the Dockerfile and server.go files as above and run :

```
cd go-docker
go mod init github.com/<your github name>/go-docker-app
go build
```

A docker image is then built with :

```
docker build -t go-web-app .
```

The following deploy.sh script creates an Azure Web Application that serves our docker image ( ensure that the word uniquename is replaced with something unique and reflective of the intended application ) :

```
#!/usr/bin/bash

# replace 'uniquename' with your own unique name

# define variables for repeated use in the current script
export RESOURCE_GROUP="uniquenameResourceGroup"
export AZURE_REGISTRY_NAME="uniquenameRegisry"
export DOCKER_IMAGE_NAME="uniquename-custom-docker-image"
export DOCKER_IMAGE_TAG="v1.0.0"
export APP_SERVICE_PLAN_NAME="uniquenameAppServicePlan"
export APP_NAME="uniquenameApplicaton" # this will need to be unique
export APP_PORT="8080" # this is the port number our docker image is configured to listen on

# build our docker image
docker build -t ${DOCKER_IMAGE_NAME} .

# create a resource group into which we can store resources pertinent to this activity
az group create --name ${RESOURCE_GROUP} --location "UK South"

# create a private registry in order to store our docker image
az acr create --name ${AZURE_REGISTRY_NAME} \
--resource-group ${RESOURCE_GROUP} --sku Basic \
--admin-enabled true


# in order to authenticate against our newly created registry, get user name and password for this
CREDENTIAL_JSON=$(az acr credential show --name ${AZURE_REGISTRY_NAME})
# the output of the credential show command will be in json format and we need to extract 
# - passwords .. value ( the first or second it doesn't matter which
# - username ( this will be the registry name )
# the following jq commands will do this for us
export REGISTRY_USERNAME=$(echo ${CREDENTIAL_JSON} | jq -r '.username')
export REGISTRY_PASSWORD=$(echo ${CREDENTIAL_JSON} | jq -r "[.passwords][0][0]|.value")

# now we authenticate against our newly created docker registry
docker login ${AZURE_REGISTRY_NAME}.azurecr.io --username ${REGISTRY_USERNAME} --password ${REGISTRY_PASSWORD}

#  tag our recently created custom docker image
docker tag ${DOCKER_IMAGE_NAME} ${AZURE_REGISTRY_NAME}.azurecr.io/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}

# push our tagged image to our private registry
docker push ${AZURE_REGISTRY_NAME}.azurecr.io/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}

# list our docker images 
az acr repository list -n ${AZURE_REGISTRY_NAME}

# create an Azure App Service Plan
az appservice plan create --name ${APP_SERVICE_PLAN_NAME} \
--resource-group ${RESOURCE_GROUP} --sku B1 --is-linux

# create the web app into which we will deploy our container
az webapp create --resource-group ${RESOURCE_GROUP} \
--plan ${APP_SERVICE_PLAN_NAME} --name ${APP_NAME} \
--deployment-container-image-name ${AZURE_REGISTRY_NAME}.azurecr.io/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}

# give the application credentials which it will need to log in to our private registry 
az webapp config container set --name ${APP_NAME} \
--resource-group ${RESOURCE_GROUP} \
--docker-custom-image-name ${AZURE_REGISTRY_NAME}.azurecr.io/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} \
--docker-registry-server-url https://${AZURE_REGISTRY_NAME}.azurecr.io \
--docker-registry-server-user ${REGISTRY_USERNAME} \
--docker-registry-server-password ${REGISTRY_PASSWORD}

# configure environment variables for the docker instance
az webapp config appsettings set --resource-group ${RESOURCE_GROUP} --name ${APP_NAME} --settings WEBSITES_PORT=${APP_PORT}
```

The application should now be available at

```
http://${APP_NAME}.azurewebsites.net
```

## References :

* [Building Docker Containers for Go Applications](https://www.callicoder.com/docker-golang-image-container-example/)
* [Tutorial: Build a custom image and run in App Service from a private registry](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-custom-docker-image)