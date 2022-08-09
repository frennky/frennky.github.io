---
layout: post
title:  "Docker"
date:   2016-09-13 19:09:30 +0100
---

# Docker

## Networking

### Bridge Network

This is the default driver and it allows application to run and communicate in standalone containers.

### Host Network

This type of network driver removes network isolation between container and host, so it uses host's network directly.

### Overlay Network

This driver allows connection between multiple Docker daemons.

## Troubleshooting

```
docker inspect ...
```

### Exit codes

| Exit code | Description |
|:----------|:------------|
| 0 | No foreground process or process completed successfully (container ran and completed) |
| 1 | Failure due to an application error running in container |
| 125 | Docker daemon has encountered an error (example: malformed command) |
| 126 | Container command cannot be invoked (example: script cannot be run in container) |
| 127 | Container command can't be found (example: wrong path) |
| 137 | Container received SIGKILL (manual container shutdown - docker kill) |
| 139 | Container received SIGEGV (memory access violation) |
| 143 | Container received SIGTERM (self-terminating or docker stop) |

### Networking

Check network information, containers using specific network etc.
```
docker network inspect ...
```
Check ports used:
```
docker inspect --format='json .NetworkSettings.Ports' ...
docker inspect --format='json .HostConfig.PortBindings' ...
```
Check DNS:
```
docker inspect --format='json .HostConfig.Dns' ...
```
