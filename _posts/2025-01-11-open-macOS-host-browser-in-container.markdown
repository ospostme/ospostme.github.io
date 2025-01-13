---
title: Expensive Try to open macOS host browser within container
date: 2025-01-11 05:15:32 Z
---

# Target

Open URL links or website links effortless within nvim, which is running in a
mac hosted docker container. It's not big deal in desktop environment, just
press `gx`, nvim will in turn trigger `vim.ui.open`,then `open` on macOS or
`xdg_open` on Linux.

So the target here is forward `xdg_open` request out to macOS host, open the
browser accordingly.

# Reference List

- [Docker open url in host browser](https://stackoverflow.com/questions/54437534/docker-open-a-url-in-the-host-browser)
- [xdg-open on macOS](https://superuser.com/questions/911735/how-do-i-use-xdg-open-from-xdg-utils-on-mac-osx)
- [xdg-open-server](https://github.com/kitsunyan/xdg-open-server)

- [--network=host option](https://stackoverflow.com/questions/43316376/what-does-network-host-option-in-docker-command-really-do)
- [socket and tcp](https://stackoverflow.com/questions/2149564/redirecting-tcp-traffic-to-a-unix-domain-socket-under-linux)
- [redirect example](https://gist.github.com/ljjjustin/585e817d1d7ce4eb75b87076e5b7aa7e)

- [tcp to unix socket](https://coderwall.com/p/c3wyzq/forwarding-tcp-traffic-to-a-unix-socket)

- [docker.host.internal](https://medium.com/@TimvanBaarsen/how-to-connect-to-the-docker-host-from-inside-a-docker-container-112b4c71bc66)
- [socat example](https://gist.github.com/mario21ic/c09f0a648130ad6a91abdde41cb011c8)

# Request for macOS

- X11 server

  - XQuartz

the XQuartz project is an open-source effort to develop a version of the X.Org
X Window System that runs on macOS. Together with supporting libraries and
applications, it forms the X11.app that Apple shipped with OS X versions 10.5
through 10.7

> Why XQuartz ?
> system clipboard, `xdg-open`, such kind of ability need X11, unless install
> X11 in docker(which increase image size significantly), otherwise you have to
> run it somewhere else.

To achieve complex graphics such as windows, decoration, visual effects, and
event handling, we’d need a complete graphics stack of necessary packages. One
of these packages is the X.Org Server. X Window system is based on the
network-transparent client-server architecture. This means that if our client –
the application window – is on one machine and our server is on another, the
client will still be interactive as long as both machines are connected.

The X protocol is responsible for the delivery of messages between a client and
a server — either locally or remotely

```

    ┌───────────┐         X11 protocol
    │ X client  │─────────────────────────┐
    └───────────┘                         │
                                          │
                                          ▼
                                ┌─────────────────┐
                                │    X Server     │
       ┌────────────────────────│    xorg         │
       │                        │    XFree86      │
       ▼                        └─┬───────────────┘
    ┌─────────────────────────────┼─────────────────────┐
    │libdrm                       │                     │
    │OpenGL ────┐                 │                     │
    │           │                 │                     │
    │           ▼                 │                     │
    │          System Interface  ◄┘                     │
    │                                                   │
    │          Direct Rendering Manager                 │
    │                                                   │
    │          Kernel                                   │
    │                                                   │
    │          GPU                                      │
    │                                                   │
    └───────────────────────────────────────────────────┘

```

- Compiling tools

  - gcc
  - xlib

- [xdg-open-server](https://github.com/kitsunyan/xdg-open-server)
  - Do not use unix socket directly, as really don't know the underlying
    difference between macOS and Linux
  - start tcp listen on host, and forward the request to xdg-open-server socket
  - wrap `xdg-open` in container, forward to host tcp listening port

The original project shares a great idea, forward `xdg-open` outside to the
host, with `socat` socket proxy to host socket file (mount on docker container).

Failed on my macOS host environment, after searching a while especially the
'AI Overview', decide to make a bit of change.

[Network-host-option](https://stackoverflow.com/questions/43316376/what-does-network-host-option-in-docker-command-really-do)

```text

AI Overview from Google

To share a socket between a Mac host and a Docker container, you generally need
to use a volume mount to expose the socket file within the container, but due to
the differences between macOS and Linux, directly sharing a Unix domain socket
is not straightforward and might require workarounds like using a proxy service
(like socat) to expose the socket over TCP. Key points to remember: Limitations
of macOS: macOS has a different Unix implementation compared to Linux, which
makes sharing Unix domain sockets directly between the host and container
challenging. Volume Mount: The primary method to share files (including socket
files) between a host and container is using the -v (or --volume) flag when
running a Docker container, but this needs to be done carefully considering the
socket type. Proxy Service (socat): To overcome the limitations of direct socket
sharing, you can use a tool like socat to act as a proxy, essentially converting
the Unix domain socket to a TCP socket accessible by the container. How to
potentially share a socket with a container (using socat):

1. Identify Socket Location: Find the location of the socket file on your Mac
  host (usually within /var/run/docker.sock).
2. Run socat in the container: Inside your container, run a command like: socat
  TCP-LISTEN:your_exposed_port UNIX-CLIENT:/var/run/docker.sock. This will
listen on a specified TCP port within the container and forward connections to
the host's Docker socket.
3. Access the socket in your application: Within your container application,
  connect to the exposed TCP port to interact with the Docker daemon. Important
considerations: Permissions: Ensure the appropriate permissions are set on the
socket file to allow the container process to access it. Network Configuration:
If you are using a custom network for your container, make sure the container is
properly connected to the host network to reach the exposed socket
```

- socat

```
nohup socat -d -d -d -lf ns-socat.log TCP-LISTEN:5678,reuseaddr,fork UNIX-CLIENT:/Users/ospost/var/run/501/xdg-open-server/socket >/dev/null 2>&1 &
```

# Request for Container

- socat wrapper

```
xdg-open

#!/bin/bash

echo "$@" | socat - TCP:host.docker.internal:${XDGOPEN_TCP_PORT}

```
