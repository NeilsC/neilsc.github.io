---
layout: post
title: "Debugging Rails Apps Running in docker-compose Using byebug"
date: 2020-11-10
categories: docker ruby
---
# The Problem
You are serving a rails application from inside Docker Compose and want to use byebug to set breakpoints and debug your app, but your breakpoints are not stopping execution and you can't interact with byebug.

# Enable interaction with your container
First you need to configure your container (via docker-compose.yml) to enable interaction via stdin and tty:
```yaml
container_name:
  ...
  stdin_open: true
  tty: true
```

For details see [here](https://docs.docker.com/compose/compose-file/#domainname-hostname-ipc-mac_address-privileged-read_only-shm_size-stdin_open-tty-user-working_dir).

# Run docker-compose in detached mode
```bash
$ docker-compose up -d
```

# View container logs
Usually `docker-compose up` will echo all of your containers' logs to stdout, but this does not happen when running in detached mode. Use `docker-compose logs` to watch the log output:
```bash
$ docker-compose logs --follow --tail=0
```

# Attach to your app to interact with byebug
Once your app is up and running in its container you can attach to it with `docker attach`:
```bash
$ docker attach <container_name>
```

Since you configured stdin and tty, you'll be able to interact with byebug.

# Stop your containers when finished
Once you're done you can stop your containers with `docker-compose stop`.
