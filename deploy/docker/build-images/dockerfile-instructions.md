# Dockerfile指令

## FROM

```text
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

`FROM`指令用于初始化一个新的构造阶段，和设定基础镜像，所以`Dockerfile`必须要以`FROM`指令开始，任意有效的镜像都可以。

* `ARG`是唯一一个可能比`FROM`更先出现的指令。
* 一个`Dockerfile`中可以有多个`FROM`指令用于构建多个镜像，或者用来表示一个构建阶段为另一个构建阶段的依赖。Docker发现`Dockerfile`中每一个`FROM`指令时，都会提交前面的指令组成的镜像，并且清空前面指令产生的任何状态。
* 可以选择在`FROM`指令中使用`AS name`来给镜像设定名字。这个名字可以被后面的`FROM`或`COPY --from=<name|index>`指令调用。
* `FROM`指令提供`tag`和`digest`选项。如果忽略他们中的一个，builder会默认`tag`的值是latest。如果builder找不到`tag`的值，会报错。
* FROM提供--platform标记用于设置这个镜像适合那个平台，例如：`lunix/amd`、`linux/arm64`、`window/arm64`。

### ARG与FROM的交互

通过在定义`FROM`指令之前定义`ARG`指令，为`FROM`指令提供变量。

```go
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

如`ARG`定义在`FROM`指令之前，所以它不属于构造阶段，所以它不能被`FROM`之后的任何指令使用。

## LABEL

## RUN

## ARG

## CMD

## EXPOSE

## ENV

## ADD

## COPY

## ENTRYPOINT

## VOLUME

## USER

## WORKDIR

## ONBUILD



