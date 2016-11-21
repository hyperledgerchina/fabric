# Fabric 开始套件

本节介绍如何使用Hyperledger fabric设置用于应用程序开发的自包含环境。该设置使用**Docker**提供受控环境和所有必要的Hyperledger fabric组件，以支持使用fabric Node.js SDK构建的Node.js应用程序和用Go编写的chaincode。

有三个Docker镜像，当运行时，将提供一个基本的网络环境。有一个映像运行单个Peer，一个运行membersrvc，一个运行你的Node.js应用程序和你的chaincode。请参阅[Application Developer's Overview](http://fabric-sdk-node.readthedocs.io/en/latest/app-overview) ，了解容器内运行的组件如何进行通信的。

入门套件附带一个示例Node.js应用程序，可立即执行和展示chaincode。入门套件将以chaincode开发者模式运行。在此模式下，在应用程序调用部署它之前，将先构建并启动chaincode。

**注意**：在网络模式下部署chaincode要求Hyperledger fabric的Node.js SDK可以访问chaincode源代码及其所有依赖项，以便正确构建部署请求。它还要求`peer`可以访问Docker守护进程，以便能够构建和部署将运行chaincode的新的Docker镜像。*这是一个更复杂的配置，不适合作为一个Hyperledger fabric介绍*。我们建议首先在chaincode开发模式下运行。

## 开始

**Note:** 这个例子是建立在MAC1.12.0 Docker的基础上的

* 必备软件安装:

  * [Docker](https://www.docker.com/products/overview)
  * docker-compose (可能是一个打包好的Docker镜像)

* 拷贝 [docker-compose.yml](https://raw.githubusercontent.com/hyperledger/fabric/master/examples/sdk/node/docker-compose.yml) 配置文件到您的本地目录:

```
   curl -o docker-compose.yml https://raw.githubusercontent.com/hyperledger/fabric/master/examples/sdk/node/docker-compose.yml
```
docker-compose环境使用三个Docker镜像。两个发布到DockerHub。但是，第三个，我们为您提供源文件自己创建，以便您可以自定义它来注入您的应用程序代码进行开发。以下 [Dockerfile](https://raw.githubusercontent.com/hyperledger/fabric/master/examples/sdk/node/Dockerfile)用于构建基本**fabric-starter-kit**镜像，并且可以用作您自己的自定义的起点。

```
   curl -o Dockerfile https://raw.githubusercontent.com/hyperledger/fabric/master/examples/sdk/node/Dockerfile
   docker build -t hyperledger/fabric-starter-kit:latest .
```

* 使用docker-compose启动Fabric网络环境。从具有上述docker-compose.yml所在的工作目录的终端开始会话，执行以下docker-compose命令。
   * 作为分离容器运行：

```
   docker-compose up -d
```
​         **备注：**查看'peer'容器的日志使用 `docker logs peer`命令

​         **译注：**-d 分离模式：作为后台来执行容器，打印新的容器名，不适用于--abort-on-container-exit（退出全部docker）命令

在前台运行并查看当前终端会话中的日志输出：

```
   docker-compose up
```

这两个命令都将启动三个Docker容器。要查看容器状态，请使用`docker ps`命令。第一次运行时，将下载Docker镜像。这可能需要10分钟或更长时间，具体取决于运行命令的系统的网络连接。

```
   docker ps
```
    您应该看到类似于以下内容的内容:

```
   CONTAINER ID    IMAGE                           COMMAND                  CREATED              STATUS              PORTS  NAMES
   bb01a2fa96ef    hyperledger/fabric-starter-kit  "sh -c 'sleep 20; /op"   About a minute ago   Up 59 seconds           starter
   ec7572e65f12    hyperledger/fabric-peer         "sh -c 'sleep 10; pee"   About a minute ago   Up About a minute          peer
   118ef6da1709    hyperledger/fabric-membersrvc   "membersrvc"             About a minute ago   Up About a minute          membersrvc
```

* start容器中起一个终端。这个这是Node.js应用程序所在的那个。

  **注意**：在使用docker-compose up命令启动网络后，请务必等待20秒，然后再执行以下命令以允许网络初始化：

```
   docker exec -it starter /bin/bash
```

* 从**starter**容器中的终端会话执行独立的Node.js应用程序。 Docker终端会话应该在名为**app.js**的示例应用程序的工作目录中(*/opt/gopath/src/github.com/hyperledger/fabric/examples/sdk/node*). 执行以下Node.js命令运行应用程序：

```
   node app
```
​     在host上的另一个终端会话中，您可以通过执行以下命令查看peer的日志（不是在上面的docker shell中，而是在真实系统的另一个新的终端中）：

```
   docker logs peer
```

* 如果您希望使用预构建的Docker镜像运行您自己的Node.js应用程序:
   * 将使用`docker-compose.yml`**starter** 配置的`volumes`标签中指定的目录，作为了本机中的程序存储目录拷贝到docker容器中。第一个路径是顶层系统目录（主机系统），第二个是在Docker容器中将创建的目录。如果你想使用不在/ Users目录下的主机位置（〜是`/ Users'），你必须将它添加到Docker首选项下的Docker文件共享中。

```yaml
  volumes:
    - ~/mytest:/user/mytest
```
* 复制或在〜/ mytest目录中创建和编辑应用程序，如**starter**容器下的`docker-compose.yml` `volumes`标签中所述。
* 运行npm在mytest目录中安装Hyperledger fabric Node.js SDK：

```
     npm install /opt/gopath/src/github.com/hyperledger/fabric-sdk-node
```
* 使用以下命令从**starter** Docker容器中运行应用程序：

```
   docker exec -it starter /bin/bash
```
   在shell中执行一次，并假设你的Node.js应用程序被称为`app.js`:

```
   cd /user/mytest
   node app
```
* 要关闭环境，请在docker-compose.yml所在的目录中执行以下**docker-compose**命令。对示例应用程序或链码的部署所做的任何更改都将丢失。只有对**starter**容器的“volumes”标签中定义的共享区域所做的更改才会保留。这将关闭每个容器，并从Docker中删除容器：

```
   docker-compose down
```
或者如果您希望保留更改并停止容器，这将在下一个up启动命令时保留更改：

```
   docker-compose kill
```

## 进一步探索

如果你愿意，这附近有一些chaincode例子.
```
   cd $GOPATH/src/github.com/hyperledger/fabric/examples/chaincode
```
