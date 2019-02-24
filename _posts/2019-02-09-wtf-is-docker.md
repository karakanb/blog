---
title: "wtf is docker?"
excerpt: "how does this docker thingy work? why should I use it?"
---

Alright, we get it, cloud is the future and we need to use containers with all the fancy tools it offers. We are going to *containerize* our app, use *container orchestration* tools for deployments, and we *have to install Docker*. 

## wtf is a container?
Remember the good old times where you used to ssh into the production server, go to the project directory, and run `git pull` to deploy your code? Before you deploy anything, in the very beginning of the life of your server, you'd install all the global dependencies for your app, `curl` most probably, then `git`, maybe the interpreter for the language you want to use, and some extensions for that as well, maybe `nginx` at some point. Once all the dependencies are installed, you'd bring your application to the server, run some commands and start it.

At this point, once you `pull`ed your code, you'd start the new version of your application, or you'd restart nginx for some changes to take effect, or whatever. This setup probably worked for a long time, until it didn't. One of the developers in your team relied on a system dependency that has a different version installed in the production server, and now your service is down. You quickly rollback your changes, but you will need to update that dependency at some point. A worse example may be the bugs caused by these kind of dependency differences in a weird place of your app, which means you probably wouldn't notice until it is too late, in other words, *already shipped*.

Consider another example, where you'd like to run multiple application on the same host, but you need them to be isolated for security reasons. You either need to move the applications into separate hosts, which is not cost efficient, or you'd run two different virtual machines in the host, which would give you the isolation but the resources will be consumed by the VMs mostly rather than your application, which is still not the best way.

These problems have existed for decades now; keeping the processes separate is a huge pain, and this caused a lot of security problems as well as inefficient setups.

## linux containers to the rescue
In this context, a container is a set of isolated processes and resources. Linux achieves this by using namespaces, which allows processes to access only namespace resources, which allows to have a process tree that is completely independent of the rest of the systems. The actual way containers work is a complex topic that I will not get into here, but overall the concept is simple: give me independent resources in a physical machine that I can do whatever I want.

## enter docker
Docker is one of the tools that used the isolated resources idea to create a set of tools that allows applications to be packaged with all the dependencies installed and ran wherever wanted. 

I can hear the thoughts like "this is the same thing as virtual machines", but the difference is, Docker containers share the same system resources, they don't have separate, dedicated resources that allows them to behave like completely independent machines, they don't need to have a full blown OS inside, and these advantages allow these containers to be pretty lightweight and efficient. A machine where you can run 2 VMs, you can run tens of Docker containers without any trouble, which means less resources = less cost = less maintenance = happy people.

Regarding the differences between the Docker containers and VMs, [here is a nice answer in StackOverflow](https://stackoverflow.com/a/16048358):
> Docker originally used LinuX Containers (LXC), but later switched to runC (formerly known as libcontainer), which runs in the same operating system as its host. This allows it to share a lot of the host operating system resources. Also, it uses a layered filesystem (AuFS) and manages networking.
>
AuFS is a layered file system, so you can have a read only part and a write part which are merged together. One could have the common parts of the operating system as read only (and shared amongst all of your containers) and then give each container its own mount for writing.
>
So, let's say you have a 1 GB container image; if you wanted to use a full VM, you would need to have 1 GB times x number of VMs you want. With Docker and AuFS you can share the bulk of the 1 GB between all the containers and if you have 1000 containers you still might only have a little over 1 GB of space for the containers OS (assuming they are all running the same OS image).
>
A full virtualized system gets its own set of resources allocated to it, and does minimal sharing. You get more isolation, but it is much heavier (requires more resources). With Docker you get less isolation, but the containers are lightweight (require fewer resources). So you could easily run thousands of containers on a host, and it won't even blink. Try doing that with Xen, and unless you have a really big host, I don't think it is possible.

Docker has two concept that is almost the same with its VM containers as the idea, an *image* and a *container*. An image is the definition of the what is going to be executed, it is just like an operating system image, and a container is the running instance of a given image.

## gimme a practical example
Docker images are defined within special text files called `Dockerfile`, and you need to define all the steps explicitly inside the Dockerfile. Here goes an example from one of my images that has Python 3.7, Chromium, Selenium + Chrome Driver and pytest in it, this is an actual image that I use for some of my acceptance test pipelines.

```dockerfile
FROM python:3.7-alpine3.9

# update apk repo
RUN echo "http://dl-4.alpinelinux.org/alpine/v3.9/main" >> /etc/apk/repositories && \
    echo "http://dl-4.alpinelinux.org/alpine/v3.9/community" >> /etc/apk/repositories

# install chromedriver
RUN apk update
RUN apk add chromium chromium-chromedriver

# install selenium
RUN pip install selenium==3.8.0 pytest pytest-xdist
```

It uses `python` base image with the tag `3.7-alpine3.9`, which is a specific version. It than puts the community repository definitions and updates the repository indexes; if you are thinking like "wtf is a community repository", give [this](https://wiki.alpinelinux.org/wiki/Enable_Community_Repository) piece a look. Then it basically uses `apk` to install `chromium` and `chromium-chromedriver`, and uses `pip` to install `selenium`, `pytest` and `pytest-xdist` which basically allows running pytest tests in parallel.

Once you have this file and named it as `Dockerfile`, you can just run this command to build your image:
```sh
docker build . -t my-image-name:my-tag
```

It will build your image, and once the build is done, you can run your image with this simple command:
```sh
docker run -it my-image-name:my-tag /bin/sh
```

This command will give you a shell inside the container, which you can use to do whatever you want. At this point, in order to understand the concept a little bit better, open another terminal session while keeping the one in the container running, and run `docker ps`, which will give you an output like this:
```sh
CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS              PORTS               NAMES
4a81c6a020c8        my-image-name:my-tag   "/bin/sh"           44 seconds ago      Up 44 seconds                           cocky_herschel
```

The one listed there is your currently running container, which you can see with the `IMAGE` column set to your new image. However, there is a small detail: if you exit the shell session inside the container by running `exit` or `CTRL+D`, your container will *die* and `docker ps` will give you an empty output.

There is a simple explanation behind this behavior; when you have executed the runner command above as `docker run -it my-image-name:my-tag /bin/sh`, you have basically told Docker to start this container with the `/bin/sh` process as the main process inside the container, which means once your process is dead, which is what happens when you exit the shell session, your container will die, simple as that.

## so, should I use this thing?
Overall, Docker allows applications to be packaged with all the dependencies inside, which simplifies the deployment process quite a bit and you get to have full reproducible environments. Docker by itself is not always enough and easy to use if you have multiple services that needs to be running together, then you can use [Docker Compose](https://docs.docker.com/compose/) which comes with Docker and is great for managing multiple containers and allowing inter-container communication and it is damn easy to get started.

I have started using Docker for my side projects around a year ago, and my experience has been a real pleasure. I am able to run isolated pipelines for my CI/CD needs, I am able to coordinate with other developers without considering platform-specific dependencies, and deploying my applications is easier than ever. There are also a lot of new stuff going on around containers, and one of the biggest hypes is around [Kubernetes](https://kubernetes.io/), which is an open-source container orchestration tool that allows you to run your containers on a cluster with autoscaling, rollouts, rollbacks and self-healing deployments, it is pretty cool.

Give this thing a try, YMMV but for most of the cases it will allow you to move faster and simplify your workflow. If not, well, `¯\_(ツ)_/¯`.



