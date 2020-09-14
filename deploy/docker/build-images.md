# 构造镜像

Docker builds images automatically by reading the instructions from a `Dockerfile` which is a text file that contains all commands, in order, needed to build a given image. A `Dockerfile` adheres to a specific format and set of instructions which you can find at [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

A Docker image consists of read-only layers each of which represents a Dockerfile instruction. The layers are stacked and each one is a delta of the changes from the previous layer. Consider this `Dockerfile`:

```bash
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

Each instruction creates one layer:

* `FROM` creates a layer from the `ubuntu:18.04` Docker image.
* `COPY` adds files from your Docker client’s current directory.
* `RUN` builds your application with `make`.
* `CMD` specifies what command to run within the container.

When you run an image and generate a container, you add a new _writable layer_ \(the “container layer”\) on top of the underlying layers. All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer.

## 标准和建议

### 创建临时容器

The image defined by your `Dockerfile` should generate containers that are as ephemeral as possible. By “ephemeral”, we mean that the container can be stopped and destroyed, then rebuilt and replaced with an absolute minimum set up and configuration.

Refer to [Processes](https://12factor.net/processes) under _The Twelve-factor App_ methodology to get a feel for the motivations of running containers in such a stateless fashion.

### 了解

When you issue a `docker build` command, the current working directory is called the _build context_. By default, the Dockerfile is assumed to be located here, but you can specify a different location with the file flag \(`-f`\). Regardless of where the `Dockerfile` actually lives, all recursive contents of files and directories in the current directory are sent to the Docker daemon as the build context.

> #### Build context example
>
> Create a directory for the build context and `cd` into it. Write “hello” into a text file named `hello` and create a Dockerfile that runs `cat` on it. Build the image from within the build context \(`.`\):
>
> ```text
> mkdir myproject && cd myproject
> echo "hello" > hello
> echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
> docker build -t helloapp:v1 .
> ```
>
> Move `Dockerfile` and `hello` into separate directories and build a second version of the image \(without relying on cache from the last build\). Use `-f` to point to the Dockerfile and specify the directory of the build context:
>
> ```text
> mkdir -p dockerfiles context
> mv Dockerfile dockerfiles && mv hello context
> docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context
> ```

Inadvertently including files that are not necessary for building an image results in a larger build context and larger image size. This can increase the time to build the image, time to pull and push it, and the container runtime size. To see how big your build context is, look for a message like this when building your `Dockerfile`:

```text
Sending build context to Docker daemon  187.8MB
```

### Pipe Dockerfile through `stdin`

Docker has the ability to build images by piping `Dockerfile` through `stdin` with a _local or remote build context_. Piping a `Dockerfile` through `stdin` can be useful to perform one-off builds without writing a Dockerfile to disk, or in situations where the `Dockerfile` is generated, and should not persist afterwards.

> The examples in this section use [here documents](http://tldp.org/LDP/abs/html/here-docs.html) for convenience, but any method to provide the `Dockerfile` on `stdin` can be used.
>
> For example, the following commands are equivalent:
>
> ```text
> echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
> ```
>
> ```text
> docker build -<<EOF
> FROM busybox
> RUN echo "hello world"
> EOF
> ```
>
> You can substitute the examples with your preferred approach, or the approach that best fits your use-case.

#### **BUILD AN IMAGE USING A DOCKERFILE FROM STDIN, WITHOUT SENDING BUILD CONTEXT**

Use this syntax to build an image using a `Dockerfile` from `stdin`, without sending additional files as build context. The hyphen \(`-`\) takes the position of the `PATH`, and instructs Docker to read the build context \(which only contains a `Dockerfile`\) from `stdin` instead of a directory:

```text
docker build [OPTIONS] -
```

The following example builds an image using a `Dockerfile` that is passed through `stdin`. No files are sent as build context to the daemon.

```text
docker build -t myimage:latest -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```

Omitting the build context can be useful in situations where your `Dockerfile` does not require files to be copied into the image, and improves the build-speed, as no files are sent to the daemon.

If you want to improve the build-speed by excluding _some_ files from the build- context, refer to [exclude with .dockerignore](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#exclude-with-dockerignore).

> **Note**: Attempting to build a Dockerfile that uses `COPY` or `ADD` will fail if this syntax is used. The following example illustrates this:
>
> ```text
> # create a directory to work in
> mkdir example
> cd example
>
> # create an example file
> touch somefile.txt
>
> docker build -t myimage:latest -<<EOF
> FROM busybox
> COPY somefile.txt .
> RUN cat /somefile.txt
> EOF
>
> # observe that the build fails
> ...
> Step 2/3 : COPY somefile.txt .
> COPY failed: stat /var/lib/docker/tmp/docker-builder249218248/somefile.txt: no such file or direc
> ```

#### **BUILD FROM A LOCAL BUILD CONTEXT, USING A DOCKERFILE FROM STDIN**

Use this syntax to build an image using files on your local filesystem, but using a `Dockerfile` from `stdin`. The syntax uses the `-f` \(or `--file`\) option to specify the `Dockerfile` to use, using a hyphen \(`-`\) as filename to instruct Docker to read the `Dockerfile` from `stdin`:

```text
docker build [OPTIONS] -f- PATH
```

The example below uses the current directory \(`.`\) as the build context, and builds an image using a `Dockerfile`that is passed through `stdin` using a [here document](http://tldp.org/LDP/abs/html/here-docs.html).

```text
# create a directory to work in
mkdir example
cd example

# create an example file
touch somefile.txt

# build an image using the current directory as context, and a Dockerfile passed through stdin
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt .
RUN cat /somefile.txt
EOF
```

#### **BUILD FROM A REMOTE BUILD CONTEXT, USING A DOCKERFILE FROM STDIN**

Use this syntax to build an image using files from a remote `git` repository, using a `Dockerfile` from `stdin`. The syntax uses the `-f` \(or `--file`\) option to specify the `Dockerfile` to use, using a hyphen \(`-`\) as filename to instruct Docker to read the `Dockerfile` from `stdin`:

```text
docker build [OPTIONS] -f- PATH
```

This syntax can be useful in situations where you want to build an image from a repository that does not contain a `Dockerfile`, or if you want to build with a custom `Dockerfile`, without maintaining your own fork of the repository.

The example below builds an image using a `Dockerfile` from `stdin`, and adds the `hello.c` file from the [“hello-world” Git repository on GitHub](https://github.com/docker-library/hello-world).

```text
docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c .
EOF
```

> **Under the hood**
>
> When building an image using a remote Git repository as build context, Docker performs a `git clone` of the repository on the local machine, and sends those files as build context to the daemon. This feature requires `git` to be installed on the host where you run the `docker build` command.

### Exclude with .dockerignore

To exclude files not relevant to the build \(without restructuring your source repository\) use a `.dockerignore` file. This file supports exclusion patterns similar to `.gitignore` files. For information on creating one, see the[.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

### Use multi-stage builds

[Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) allow you to drastically reduce the size of your final image, without struggling to reduce the number of intermediate layers and files.

Because an image is built during the final stage of the build process, you can minimize image layers by [leveraging build cache](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache).

For example, if your build contains several layers, you can order them from the less frequently changed \(to ensure the build cache is reusable\) to the more frequently changed:

* Install tools you need to build your application
* Install or update library dependencies
* Generate your application

A Dockerfile for a Go application could look like:

```text
FROM golang:1.11-alpine AS build

# Install tools required for project
# Run `docker build --no-cache .` to update dependencies
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

# List project dependencies with Gopkg.toml and Gopkg.lock
# These layers are only re-built when Gopkg files are updated
COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
# Install library dependencies
RUN dep ensure -vendor-only

# Copy the entire project and build it
# This layer is rebuilt when a file changes in the project directory
COPY . /go/src/project/
RUN go build -o /bin/project

# This results in a single layer image
FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
```

### Don’t install unnecessary packages

To reduce complexity, dependencies, file sizes, and build times, avoid installing extra or unnecessary packages just because they might be “nice to have.” For example, you don’t need to include a text editor in a database image.

### Decouple applications

Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers. For instance, a web application stack might consist of three separate containers, each with its own unique image, to manage the web application, database, and an in-memory cache in a decoupled manner.

Limiting each container to one process is a good rule of thumb, but it is not a hard and fast rule. For example, not only can containers be [spawned with an init process](https://docs.docker.com/engine/reference/run/#specify-an-init-process), some programs might spawn additional processes of their own accord. For instance, [Celery](http://www.celeryproject.org/) can spawn multiple worker processes, and [Apache](https://httpd.apache.org/) can create one process per request.

Use your best judgment to keep containers as clean and modular as possible. If containers depend on each other, you can use [Docker container networks](https://docs.docker.com/network/) to ensure that these containers can communicate.

### Minimize the number of layers

In older versions of Docker, it was important that you minimized the number of layers in your images to ensure they were performant. The following features were added to reduce this limitation:

* Only the instructions `RUN`, `COPY`, `ADD` create layers. Other instructions create temporary intermediate images, and do not increase the size of the build.
* Where possible, use [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/), and only copy the artifacts you need into the final image. This allows you to include tools and debug information in your intermediate build stages without increasing the size of the final image.

### Sort multi-line arguments

Whenever possible, ease later changes by sorting multi-line arguments alphanumerically. This helps to avoid duplication of packages and make the list much easier to update. This also makes PRs a lot easier to read and review. Adding a space before a backslash \(`\`\) helps as well.

Here’s an example from the [`buildpack-deps` image](https://github.com/docker-library/buildpack-deps):

```text
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

### Leverage build cache

When building an image, Docker steps through the instructions in your `Dockerfile`, executing each in the order specified. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse, rather than creating a new \(duplicate\) image.

If you do not want to use the cache at all, you can use the `--no-cache=true` option on the `docker build`command. However, if you do let Docker use its cache, it is important to understand when it can, and cannot, find a matching image. The basic rules that Docker follows are outlined below:

* Starting with a parent image that is already in the cache, the next instruction is compared against all child images derived from that base image to see if one of them was built using the exact same instruction. If not, the cache is invalidated.
* In most cases, simply comparing the instruction in the `Dockerfile` with one of the child images is sufficient. However, certain instructions require more examination and explanation.
* For the `ADD` and `COPY` instructions, the contents of the file\(s\) in the image are examined and a checksum is calculated for each file. The last-modified and last-accessed times of the file\(s\) are not considered in these checksums. During the cache lookup, the checksum is compared against the checksum in the existing images. If anything has changed in the file\(s\), such as the contents and metadata, then the cache is invalidated.
* Aside from the `ADD` and `COPY` commands, cache checking does not look at the files in the container to determine a cache match. For example, when processing a `RUN apt-get -y update` command the files updated in the container are not examined to determine if a cache hit exists. In that case just the command string itself is used to find a match.

Once the cache is invalidated, all subsequent `Dockerfile` commands generate new images and the cache is not used.

## Docker命令







**参考资料：**

* \*\*\*\*[**Best practices for writing Dockerfiles**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)\*\*\*\*

