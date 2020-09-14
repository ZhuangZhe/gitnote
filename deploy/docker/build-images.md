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

### 了解构建环境

When you issue a `docker build` command, the current working directory is called the _build context_. By default, the Dockerfile is assumed to be located here, but you can specify a different location with the file flag \(`-f`\). Regardless of where the `Dockerfile` actually lives, all recursive contents of files and directories in the current directory are sent to the Docker daemon as the build context.

> #### 构建环境例子
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

### 使用`.dockerignore`排除镜像中不需要的文件

To exclude files not relevant to the build \(without restructuring your source repository\) use a `.dockerignore` file. This file supports exclusion patterns similar to `.gitignore` files. For information on creating one, see the[.dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file).

### 使用多阶段构建

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

### 不安装不必要的包

To reduce complexity, dependencies, file sizes, and build times, avoid installing extra or unnecessary packages just because they might be “nice to have.” For example, you don’t need to include a text editor in a database image.

### 解耦应用

Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers. For instance, a web application stack might consist of three separate containers, each with its own unique image, to manage the web application, database, and an in-memory cache in a decoupled manner.

Limiting each container to one process is a good rule of thumb, but it is not a hard and fast rule. For example, not only can containers be [spawned with an init process](https://docs.docker.com/engine/reference/run/#specify-an-init-process), some programs might spawn additional processes of their own accord. For instance, [Celery](http://www.celeryproject.org/) can spawn multiple worker processes, and [Apache](https://httpd.apache.org/) can create one process per request.

Use your best judgment to keep containers as clean and modular as possible. If containers depend on each other, you can use [Docker container networks](https://docs.docker.com/network/) to ensure that these containers can communicate.

### 最小化layer的数量

In older versions of Docker, it was important that you minimized the number of layers in your images to ensure they were performant. The following features were added to reduce this limitation:

* Only the instructions `RUN`, `COPY`, `ADD` create layers. Other instructions create temporary intermediate images, and do not increase the size of the build.
* Where possible, use [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/), and only copy the artifacts you need into the final image. This allows you to include tools and debug information in your intermediate build stages without increasing the size of the final image.

### 排序多行参数

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

### 利用构建镜像的缓存

When building an image, Docker steps through the instructions in your `Dockerfile`, executing each in the order specified. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse, rather than creating a new \(duplicate\) image.

If you do not want to use the cache at all, you can use the `--no-cache=true` option on the `docker build`command. However, if you do let Docker use its cache, it is important to understand when it can, and cannot, find a matching image. The basic rules that Docker follows are outlined below:

* Starting with a parent image that is already in the cache, the next instruction is compared against all child images derived from that base image to see if one of them was built using the exact same instruction. If not, the cache is invalidated.
* In most cases, simply comparing the instruction in the `Dockerfile` with one of the child images is sufficient. However, certain instructions require more examination and explanation.
* For the `ADD` and `COPY` instructions, the contents of the file\(s\) in the image are examined and a checksum is calculated for each file. The last-modified and last-accessed times of the file\(s\) are not considered in these checksums. During the cache lookup, the checksum is compared against the checksum in the existing images. If anything has changed in the file\(s\), such as the contents and metadata, then the cache is invalidated.
* Aside from the `ADD` and `COPY` commands, cache checking does not look at the files in the container to determine a cache match. For example, when processing a `RUN apt-get -y update` command the files updated in the container are not examined to determine if a cache hit exists. In that case just the command string itself is used to find a match.

Once the cache is invalidated, all subsequent `Dockerfile` commands generate new images and the cache is not used.

## Docker命令

### FROM

```text
FROM [--platform=<platform>] <image> [AS <name>]
```

Or

```text
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```

Or

```text
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

The `FROM` instruction initializes a new build stage and sets the [_Base Image_](https://docs.docker.com/glossary/#base_image) for subsequent instructions. As such, a valid `Dockerfile` must start with a `FROM` instruction. The image can be any valid image – it is especially easy to start by **pulling an image** from the [_Public Repositories_](https://docs.docker.com/engine/tutorials/dockerrepos/).

* `ARG` is the only instruction that may precede `FROM` in the `Dockerfile`. See [Understand how ARG and FROM interact](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact).
* `FROM` can appear multiple times within a single `Dockerfile` to create multiple images or use one build stage as a dependency for another. Simply make a note of the last image ID output by the commit before each new `FROM` instruction. Each `FROM` instruction clears any state created by previous instructions.
* Optionally a name can be given to a new build stage by adding `AS name` to the `FROM` instruction. The name can be used in subsequent `FROM` and `COPY --from=<name|index>` instructions to refer to the image built in this stage.
* The `tag` or `digest` values are optional. If you omit either of them, the builder assumes a `latest` tag by default. The builder returns an error if it cannot find the `tag` value.

The optional `--platform` flag can be used to specify the platform of the image in case `FROM` references a multi-platform image. For example, `linux/amd64`, `linux/arm64`, or `windows/amd64`. By default, the target platform of the build request is used. Global build arguments can be used in the value of this flag, for example, [automatic platform ARGs](https://docs.docker.com/engine/reference/builder/#automatic-platform-args-in-the-global-scope) allow you to force a stage to native build platform \(`--platform=$BUILDPLATFORM`\), and use it to cross-compile to the target platform inside the stage.

#### Understand how ARG and FROM interact

`FROM` instructions support variables that are declared by any `ARG` instructions that occur before the first `FROM`.

```text
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

An `ARG` declared before a `FROM` is outside of a build stage, so it can’t be used in any instruction after a `FROM`. To use the default value of an `ARG` declared before the first `FROM` use an `ARG` instruction without a value inside of a build stage:

```text
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

### LABEL

You can add labels to your image to help organize images by project, record licensing information, to aid in automation, or for other reasons. For each label, add a line beginning with `LABEL` and with one or more key-value pairs. The following examples show the different acceptable formats. Explanatory comments are included inline.

> Strings with spaces must be quoted **or** the spaces must be escaped. Inner quote characters \(`"`\), must also be escaped.

```text
# Set one or more individual labels
LABEL com.example.version="0.0.1-beta"
LABEL vendor1="ACME Incorporated"
LABEL vendor2=ZENITH\ Incorporated
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```

An image can have more than one label. Prior to Docker 1.10, it was recommended to combine all labels into a single `LABEL`instruction, to prevent extra layers from being created. This is no longer necessary, but combining labels is still supported.

```text
# Set multiple labels on one line
LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"
```

The above can also be written as:

```text
# Set multiple labels at once, using line-continuation characters to break long lines
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```

See [Understanding object labels](https://docs.docker.com/config/labels-custom-metadata/) for guidelines about acceptable label keys and values. For information about querying labels, refer to the items related to filtering in [Managing labels on objects](https://docs.docker.com/config/labels-custom-metadata/#manage-labels-on-objects). See also [LABEL](https://docs.docker.com/engine/reference/builder/#label) in the Dockerfile reference.

### RUN









**参考资料：**

* \*\*\*\*[**Best practices for writing Dockerfiles**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)\*\*\*\*

