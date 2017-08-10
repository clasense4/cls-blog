+++
title = "How to Build Minimal Docker Go Images"
author = "Fajri Abdillah"
share = true
tags = ["go", "golang", "docker", "minimal", "images"]
draft = false
menu = ""
image = "images/matt-popovich-604362.jpg"
slug = "how-to-build-minimal-docker-go-images"
date = "2017-08-08T08:36:00+07:00"
comments = true
+++

<!--more-->

## Background

Recently in Ebizu, I have task to build global deployment system. We have running NodeJS on Elastic Beanstalk. But it is below our expectation. And suddenly I want to try golang. After some googling, I decide to use ECS (EC2 Container Service). After some googling, mostly the tutorial is not what I expected. My compiled golang web app is below than 10MB. But, I want to dockerize my web app as minimal as possible.

After some googling, I found tutorial on [Codeship](https://blog.codeship.com/building-minimal-docker-containers-for-go-applications/), and cool blog of [Ta-Ching](https://tachingchen.com/blog/Building-Minimal-Docker-Image-for-Go-Applications/). I want to fill the missing part of those tutorial. And I use Mac to do this tutorial.

## Go(lang) Web Application

This is Go & Docker Version.

```
% go version
go version go1.8.3 darwin/amd64
% docker --version
Docker version 17.06.0-ce, build 02c1d87
```

And we create `server.go` file.

```
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello Golang")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

As you can see, it is straight forward, to run it, type `go run server.go` and open `http://localhost:8080`.

## Compile

First, let's try to compile our web app to binary file.

```
% go build -o server server.go
% ls -ahl
-rwxr-xr-x   1 fajriabdillah  staff   5.6M Aug  8 20:57 server
-rw-r--r--   1 fajriabdillah  staff   232B Aug  8 20:32 server.go
```

Try to running our web app, `./server` and open `http://localhost:8080`. It should be the same as before.

## Dockerize Our Application

Everything seems correct, and what we need todo is build our `Dockerfile`.

```
FROM scratch
ADD server /
CMD ["/server"]
```

And build our docker images.

```
% docker build -t server .
...
% docker images
REPOSITORY  TAG                 IMAGE ID            CREATED              SIZE
server      latest              3ae1a0189628        About a minute ago   5.86MB
```

## Run

It seems our images is fine, let's trying to run it.

```
% docker run --rm -p 8080:8080 server
standard_init_linux.go:187: exec user process caused "exec format error"
```

Whoops, it is not correct. And I investigate with `file` command.

```
% file server
server: Mach-O 64-bit executable x86_64
```

## Re-Compile

After some time, I think my build command is wrong, let's recompile our web app.

```
% CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o server_linux server.go
% ls -ahl
-rw-r--r--   1 fajriabdillah  staff    46B Aug  8 21:09 Dockerfile
-rwxr-xr-x   1 fajriabdillah  staff   5.6M Aug  8 21:08 server
-rw-r--r--   1 fajriabdillah  staff   232B Aug  8 20:32 server.go
-rwxr-xr-x   1 fajriabdillah  staff   5.6M Aug  8 21:23 server_linux
```

Let's investigate our new build

```
% file server_linux
server_linux: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```

It is different than before, if we try to run our new build.

```
% ./server_linux
zsh: exec format error: ./server_linux
```

## Re-Dockerize

Our new `Dockerfile` will be like this

```
FROM scratch
ADD server_linux /
CMD ["/server_linux"]
```

And build our docker images.

```
% docker build -t server_linux .
Sending build context to Docker daemon  17.58MB
Step 1/3 : FROM scratch
 --->
Step 2/3 : ADD server_linux /
 ---> b5b114a83921
Removing intermediate container 150b9a809468
Step 3/3 : CMD /server_linux
 ---> Running in 9d41b2f06598
 ---> ed3ca8088656
Removing intermediate container 9d41b2f06598
Successfully built ed3ca8088656
Successfully tagged server_linux:latest
% docker images
REPOSITORY    TAG                 IMAGE ID            CREATED             SIZE
server_linux  latest              ed3ca8088656        3 minutes ago       5.86MB
server        latest              3ae1a0189628        13 minutes ago      5.86MB
```

## Re-Run

Let's re-run our web app

```
% docker run --rm -p 8080:8080 server_linux
```

It does not give any error message, great, let's open `http://localhost:8080`.

## Conclusion

We achieve our goal, our docker images is really minimal, below than 10MB. Ofcourse it will be faster to push on ECR (EC2 Container Registry) or dockerhub.
