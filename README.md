# ukiyo

## Idea incubation

- ukiyo will act as a watcher for docker containers. It will run alongside with the other running containers and will be responsible for automatic updates. Updates will be based on push based model compared to existing solutions such as [watchtower](https://github.com/containrrr/watchtower) and [ouroboros](https://github.com/pyouroboros/ouroboros)

- Push events will be recived from ukiyo via webhooks. Docker registries provide webhooks to subscribe and listen to image changes. Locally running images will change only after such an event is received by ukiyo.

- Pull based model can be implemented as an optional way of updating the running containers.

## Components

- Container manager
- Push manager (webhooks configuration) 
- Scheduler 
- OPTIONAL - Pull based update implementation

## Execution modes

Two modes of execution

- As a container running alongside other containers (Should mount docker.sock to run docker commands inside the ukiyo docker container)
- As a standalone executable

## Language

- Go lang

### Go style guide

- https://github.com/rajikaimal/go-styleguide

## Dev Setup Guide

Setup docker
```sh
$ go mod init ukiyo
$ go mod tidy
```

Create docker binary file
```sh	
$ set GOARCH=amd64
$ set GOOS=linux
$ go build -ldflags="-s -w" -o ukiyo main.go
```

Docker build command
```sh
$ docker build -f Dockerfile -t agentukiyo/ukiyo:tag .
$ docker push agentukiyo/ukiyo:tag
```

Run ukiyo agent
```sh
$ docker run -p 8080:8080 \
     -- name ukiyo \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v /home/reinventor/dbs:/dbs \
     agentukiyo/ukiyo:01
```

Add webhook to your dockerhub repository
```
http:{serverIP}:8080/ukiyo-web-hook
```

Add your own docker registy details
```
http:{serverIP}:8080/save-container-access-keys
{
     "username":"docker registry name"
     "desc":"docker registry description"
     "accessToken":"docker registry accesstoken"
     "email":"your email"
     "serverAddress":"http://docker.io/v1"
}

http:{serverIP}:8080/edit-container-access-keys
{
     "username":"docker registry name"
     "desc":"edited description"
     "accessToken":"new docker registry accesstoken"
     "email":"new email"
     "serverAddress":"http://docker.io/v1"
}

http://{serverIP}:8080/delete-container-access-keys/{registryname}
```

Add your deployment details
```
http://{serverIP}:8080/save-repository-scheduled-time
{
     "name": "repository name",
     "bindingPort": 
     [{
        "exportPort": "8180",
        "internalPort": "80"
      },
      {
         "exportPort": "443",
         "internalPort": "443"
      }],
      "scheduledAt": "1555438658",
      "scheduledDowntime": false
}
        
http://{serverIP}:8080/edit-repository-scheduled-time
{
     "name": "repository name",
     "bindingPort": 
     [{
        "exportPort": "8180",
        "internalPort": "80"
     }],
     "scheduledAt": "1555438658",
     "scheduledDowntime": false
}

http://{serverIP}:8080/remove-repository-scheduled-time/{repositoryname}
```

List your deployment details   
```
http://{serverIP}:8080/get-container-history/1 
        
http://{serverIP}:8080/get-container-history-by-name/{repositoryname}
```
