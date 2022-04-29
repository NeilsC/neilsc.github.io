---
layout: post
title: "HOWTO: Build and Run restic's Rest Server on Raspberry PI OS"
date: 2022-04-28
tags: docker self-hosting
---
In the ongoing effort to re-Google my life, I've been doing quite a lot of work on my self-hosting setup. So far, I have FileRun (Dropbox replacement), Plex Media Server, and PhotoPrism (Google Photos alternative) up and running. All of this is running on a Debian on an old laptop, orchestrated by docker-compose.

When self-hosting, backups are a huge consideration, and it's important to get it right. You have no one else to blame when a hard drive fails and you loose your data. When it comes to backups on Linux, I'm really liking [restic](https://restic.readthedocs.io/en/stable/index.html). It's easy to install and run, the documentation is great, and it backs up to a wide variety of targets. One of these targets is restic's own [REST server](https://restic.readthedocs.io/en/stable/index.html).

My plan is to run the REST server on a Raspberry Pi, with a 5 TB USB drive attached to store backups locally. For off-site backups, I'm testing out [Wasabi](https://wasabi.com/), a cheap S3 compatible storage provider.

On to the fun!

# Install restic REST Server on Raspberry Pi OS

rest-server installation looks pretty easy using Docker. I was able to install Docker with no issue by following the [instructions](https://docs.docker.com/engine/install/debian/) for Debian (Raspberry Pi OS is based on Debian). However, when I attempted to run the rest-server image I ran into some issues.

{% highlight bash %}
> docker pull restic/rest-server:latest
> docker run -p 8000:8000 -v /my/data:/data --name rest_server restic/rest-server
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm/v7) and no specific platform was requested
standard_init_linux.go:228: exec user process caused: exec format error
{% endhighlight %}

Looks like we won't be able to run the provided amd64 image on the Pi, so let's try to build the image for Arm. After a bit of searching I found [some info](https://www.docker.com/blog/multi-arch-images/) about building multi-arch images using `docker buildx`.

{% highlight bash %}
> git clone https://github.com/restic/rest-server.git
> cd rest-server
> docker buildx build --platform linux/arm/v7 -t rest-server:arm --load .
{% endhighlight %}

This almost worked, until this error presented itself:

{% highlight bash %}
#0 18.18 go: missing Git command. See https://golang.org/s/gogetcmd
#0 18.18 error obtaining VCS status: exec: "git": executable file not found in $PATH
error: failed to solve: process "/bin/sh -c go build -o rest-server ./cmd/rest-server" did not complete successfully: exit code: 1
{% endhighlight %}

Not exactly sure why this happens, but git is missing from the build environment. I was able to resolve this with a small addition to the Dockerfile:

{% highlight bash %}
> git diff
diff --git a/Dockerfile b/Dockerfile
index 2debe41..a370597 100644
--- a/Dockerfile
+++ b/Dockerfile
@@ -2,6 +2,8 @@ FROM golang:alpine AS builder
 
 ENV CGO_ENABLED 0
 
+RUN apk update && apk add --no-cache git
+
 COPY . /build
 WORKDIR /build
 RUN go build -o rest-server ./cmd/rest-server
{% endhighlight %}

After that, the `docker buildx build` command worked and running the new ARM container was a breeze:

{% highlight bash %}
> docker run -dit --restart unless-stopped -p 8000:8000 -v /mnt/samsung_usb/rest_server:/data --name rest_server rest-server:arm
> curl http://localhost:8000
Unauthorized
{% endhighlight %}

Now we're off to the races! More to come.
