# devops_sandbox

A repo for exploring devops tools/workflows/deployments.  It is mostly a
  personal project to help accelerate my software development, but if/when
  things become useful I will more properly package this as an open source
  software release and adopt more best practices so that my software plays well
  with others'.

[This video is nice for understanding DevOps generally](https://www.youtube.com/watch?v=1xo-0gCVhTU)

# docker

Please consult your system's relevant documentation for details on deploying
docker.  For the purposes of this repo, we will be deploying on Arch Linux,
which is a very developer-friendly distribution of Linux.  It trivially* has
the ability to deploy very bespoke dependencies and is designed in such a way that it plays
extremely well with containers.  You can build exactly what you need and
nothing else, even if what you need is legacy _and_ the latest stable release.
This complements the work docker does, if you happen to build your containers
on top of Arch images.  It also enables you to build custom infrastructure
relatively simply within your own slight variation on the arch linux official
releases.  Arch Linux ARM is an example of a successful downstream project that
builds off of the excellent work of the Arch community.

\*(relative to competitors, the closest of which I would argue is FreeBSD, though
they lack as robust a community/adoption imo)

Now that I've preached a bit on Arch, let's get to docker:

WARNING: for the sake of convenience, I will not be specifying when you are
running as user or as root.  This initial version of the repo is targeted more
at developers, who I assume know when you should run as root or as a user.  I
welcome issues and clarifying pull-requests.

```
pacman -Syu docker #note, the development version is available in the AUR, this is the stable release
docker info #this will serve as a basic check that the install worked
```

## helloworld

Because Arch is well-designed, if you attempt to do `docker run helloworld`, it
will fail.  The Arch Linux community has made the correct decision to have
their infrastructure and critical components default to more secure practices.
This includes things like ports needing to be explicitly opened instead of
being open by default.  So do this:

```
/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
```

This opens the remote API on port 4243 and gives the host machine access via
terminal commands thanks to the socket connection.  As docker access is almost
always root-equivalent without robust hardening, you should always be this
careful with docker deployment.  Even for little projects on your home machine.
You want to mess up on them, not on your production projects!

The above command is good to know and you should understand it. But you
probably don't want to manually deploy your docker daemon.  That's what systemd
(or similar) is for.  So to do the equivalent and have it managed by systemd,
do:
```
/etc/systemd/system/docker.service.d/override.conf
---

[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock
```

Sorry, but docker helloworld will still fail.  Because you need to login (get account here: 
hub.docker.com/) . Do
```
docker login
```

You hopefully get a warning like this.  Take it into consideration:
```
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
```

At a minimum, make sure only root has permissions.

We are now ready to play with docker. Check it out:
```
docker pull hello-world
docker run hello-world
```

I had a few issues come up, but they weren't too bad to figure out.  This
  should be enough to get up and playing with docker, which will help build
  your intuition for more complex production deployments.

Upon success, you should see something like this:
```
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## slightly less trivial example

```
docker pull tutum/hello-world  # get a more complicated hello-world, I recommend checking the github
docker run -d -p 3000:80 tutum/hello-world #-p: bind port on my machine to a port within docker 3000:80
                                           #-d: detach, similar to &, can make analogy to running in background
```

You should now see a hello world web page with your container id at localhost:3000

At this point, you are ready to learn docker the only way you learn anything:
go use it to solve problems!

## docker quick ref

```
docker ps -a #ps, but for docker
docker run <thinng>
docker stop <thing>
docker login
docker pull <app> #get images (or containers?) from registry
docker rm <thing> #remove container (and maybe other objects?)
docker rmi <image> #remove image, you should always remove unused images!
```


# k8s



