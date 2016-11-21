# APIs - 命令行，REST和Node.js

## 概述

本文介绍3种可以和peer节点交互的API: 

1. [命令行](#命令行)
2. [REST API](#rest-api)
3. [Node.js应用程序](#nodejs应用程序)
   * [使用Swagger](#使用swagger-js插件)
   * [弹珠示例应用程序](#弹珠示例应用)
   * [商业票据示例应用程序](#商业票据示例应用程序)

**注意:** 如果你正在安全模式下使用API，请在继续下文之前先回顾[安全设置说明](https://github.com/hyperledgerchina/fabric_zh_CN/blob/v0.6_zh_CN/zh_CN/Setup/Chaincode-setup.md#security-setup-optional)。 

## 命令行

执行以下命令，查看现在可用的命令行命令: 

    cd /opt/gopath/src/github.com/hyperledger/fabric
    build/bin/peer

能看到和下面示例类似的输出: (**注意:** 下面的根命令peer是硬编码在[main.go](https://github.com/hyperledger/fabric/blob/v0.6/peer/main.go)中的。现在的构建会创建一个*peer*可执行文件。)

```
    Usage:
      peer [flags]
      peer [command]

    Available Commands:
      version     Print fabric peer version.
      node        node specific commands.
      network     network specific commands.
      chaincode   chaincode specific commands.
      help        Help about any command

    Flags:
      -h, --help[=false]: help for peer
          --logging-level="": Default logging level and overrides, see core.yaml for full syntax
          --test.coverprofile="coverage.cov": Done
      -v, --version[=false]: Show current version number of fabric peer server


    Use "peer [command] --help" for more information about a command.

```

如上面所示，`peer`命令支持几个子命令和参数标记。为了方便在脚本程序中调用，`peer`命令执行失败时会返回一个非0值。在命令执行成功时，子命令会在 **标准输出** 上生成下表的结果: 

命令 | 标准输出上的成功结果
--- | ---
`version`          | [core.yaml](https://github.com/hyperledger/fabric/blob/v0.6/peer/core.yaml)中定义的`peer.version`
`node start`       | N/A
`node status`      | [StatusCode](https://github.com/hyperledger/fabric/blob/v0.6/protos/server_admin.proto#L36)的字符串形式
`node stop`        | [StatusCode](https://github.com/hyperledger/fabric/blob/v0.6/protos/server_admin.proto#L36)的字符串形式
`network login`    | N/A
`network list`     | 在区块链网络上连接到该peer节点的其他peer节点
`chaincode deploy` | chaincode容器名(一个hash)，后续的`chaincode invoke`和`chaincode query`会用到
`chaincode invoke` | 事务的ID(UUID)
`chaincode query`  | 默认地，查询结果被格式化成字符串。命令行选项也支持将查询结果以原始字节的形式返回(-r，--raw)，或者格式化成16进制的字节码(-x，--hex)。如果查询结果是空的,就没有输出。


### 部署Chaincode

部署操作会为chaincode创建docker镜像，随后会把chaincode包部署到验证节点中。如下例: 

`peer chaincode deploy -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'`

chaincode deploy命令会返回chaincode的标识符(一个hash)，这个标识符会被后续的`chaincode invoke`和`chaincode query`命令使用，用于分辨已部署的chaincode。

如果开启了安全设置，要在命令里加入参数-u，传入已登录用户的用户名，如下: 

`peer chaincode deploy -u jim -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'`

**注意:** 如果GOPATH环境变量包含多个路径，chaincode必须在第一个路径下，否则部署操作会失败。

### 验证结果

可以在命令行中调用REST服务`/chain`，来验证包含最新事务的区块是否已经被添加到区块链中。IP地址要指定到一个验证节点或非验证节点上。在下面的例子中，172.17.0.2是一个验证节点或非验证节点的IP地址，7050是REST接口的端口号。REST服务监听的地址和端口号配置在[core.yaml](https://github.com/hyperledger/fabric/blob/v0.6/peer/core.yaml)中。

`curl 172.17.0.2:7050/chain`

下面是响应内容: 

```
{
    "height":1,
    "currentBlockHash":"4Yc4yCO95wcpWHW2NLFlf76OGURBBxYZMf3yUyvrEXs5TMai9qNKfy9Yn/=="
}
```

返回的BlockchainInfo消息被定义在[fabric.proto](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L96)。

```
message BlockchainInfo {
    uint64 height = 1;
    bytes currentBlockHash = 2;
    bytes previousBlockHash = 3;
}
```

使用REST服务`/chain/blocks/{Block}`验证一个块是否在区块链中。和上面一样，指定一个验证节点或非验证节点的IP地址以及7050端口号。

`curl 172.17.0.2:7050/chain/blocks/0`

返回的Block消息结构被定义在[fabric.proto](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L84)。

```
message Block {
    uint32 version = 1;
    google.protobuf.Timestamp timestamp = 2;
    repeated Transaction transactions = 3;
    bytes stateHash = 4;
    bytes previousBlockHash = 5;
    bytes consensusMetadata = 6;
    NonHashData nonHashData = 7;
}
```

返回的Block结构的消息: 

```
{
    "transactions":[{
        "type":1,
        "chaincodeID": {
            "path":"github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02"
        },
        "payload":"ClwIARJYCk9naXRod...",
        "uuid":"abdcec99-ae5e-415e-a8be-1fca8e38ba71"
    }],
    "stateHash":"PY5YcQRu2g1vjiAqHHshoAhnq8CFP3MqzMslcEAJbnmXDtD+LopmkrUHrPMOGSF5UD7Kxqhbg1XUjmQAi84paw=="
}
```

还有一些其他的命令行命令的信息，请看[protocol specification](https://github.com/hyperledger/fabric/blob/v0.6/docs/protocol-spec.md)的6.3部分。

## REST API

你可以选择任何工具来使用REST API。比如，curl命令或基于浏览器的客户端－火狐的Rest客户端或Chrome的Postman。同样地，也可以通过[Swagger](http://swagger.io/)直接触发REST请求。可以直接用Swagger服务，或者，你喜欢的话，也可以按照后面的[介绍](#to-set-up-swagger-ui)在本地安装Swagger。

**注意:** REST接口默认端口号是`7050`。可以在[core.yaml](https://github.com/hyperledger/fabric/blob/v0.6/peer/core.yaml)中修改`rest.address`。如果使用Vagrant，REST端口的映射定义在[Vagrantfile](https://github.com/hyperledger/fabric/blob/v0.6/devenv/Vagrantfile)。

**构建测试区块链时注意** 如果要在本地测试REST API，可以运行TestServerOpenchain_API_GetBlockCount测试来构建一个测试区块链，然后重启peer进程。这个测试实现在[api_test.go](https://github.com/hyperledger/fabric/blob/v0.6/core/rest/api_test.go)，它将创建一个5个区块的区块链。

```
    cd /opt/gopath/src/github.com/hyperledger/fabric/core/rest
    go test -v -run TestServerOpenchain_API_GetBlockCount
```

### REST服务 

如果用Swagger学习REST API，请点[这里](https://github.com/hyperledger/fabric/blob/v0.6/core/rest/rest_api.json)。可以直接把这个服务描述文件上传到Swagger服务中，或者，你喜欢的话，也可以按照后面的[介绍](#to-set-up-swagger-ui)在本地安装Swagger。

* [区块](#区块)
  * GET /chain/blocks/{Block}
* [区块链](#区块链)
  * GET /chain
* [Chaincode](#chaincode)
    * POST /chaincode
* [网络](#网络)
  * GET /network/peers
* [注册](#注册)
  * POST /registrar
  * DELETE /registrar/{enrollmentID}
  * GET /registrar/{enrollmentID}
  * GET /registrar/{enrollmentID}/ecert
  * GET /registrar/{enrollmentID}/tcert
* [事务](#事务)
    * GET /transactions/{UUID}

#### 区块

* **GET /chain/blocks/{Block}**

使用区块的API从区块链中获取各个区块的内容。返回的Block消息结构定义在[fabric.proto](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L84)。

```
message Block {
    uint32 version = 1;
    google.protobuf.Timestamp Timestamp = 2;
    repeated Transaction transactions = 3;
    bytes stateHash = 4;
    bytes previousBlockHash = 5;
}
```

#### 区块链

* **GET /chain**

使用链的API获取区块链的当前状态。返回的BlockchainInfo消息定义在[fabric.proto](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L96)。

```
message BlockchainInfo {
    uint64 height = 1;
    bytes currentBlockHash = 2;
    bytes previousBlockHash = 3;
}
```

#### Chaincode

* **POST /chaincode**

使用REST服务`/chaincode`来部署，调用和查询一个指定的chaincode。这个服务实现了[JSON RPC 2.0规范](http://www.jsonrpc.org/specification)，请求中的`method`字段用来表示要调用的chaincode操作。现在支持`deploy`，`invoke`和`query`。

`/chaincode`实现了[JSON RPC 2.0规范](http://www.jsonrpc.org/specification)，所以，请求内容中必须包含`jsonrpc`字段，`method`字段，以及我们例子中的`params`字段。客户端如果想收到请求的响应，还需要在请求内容中添加`id`字段，因为服务端会把没有`id`的请求假设成一个通知，就不生成响应了。

接下来的这些例子可能会用于部署，调用和查询一个示例chaincode。在部署一个chaincode时，用[ChaincodeSpec](https://github.com/hyperledger/fabric/blob/v0.6/protos/chaincode.proto#L60)来识别部署请求中的chaincode信息。

非安全模式下的chaincode部署请求: 

```
POST host:port/chaincode

{
  "jsonrpc": "2.0",
  "method": "deploy",
  "params": {
    "type": 1,
    "chaincodeID":{
        "path":"github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02"
    },
    "ctorMsg": {
        "args":["init", "a", "1000", "b", "2000"]
    }
  },
  "id": 1
}
```

如果在安全模式下部署chaincode，就需要在上面的请求里添加`secureContext`元素，其值是已注册且登录用户的registrationID。

安全模式下的chaincode部署请求(添加了`secureContext`元素): 

```
POST host:port/chaincode

{
  "jsonrpc": "2.0",
  "method": "deploy",
  "params": {
    "type": 1,
    "chaincodeID":{
        "path":"github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02"
    },
    "ctorMsg": {
        "args":["init", "a", "1000", "b", "2000"]
    },
    "secureContext": "lukas"
  },
  "id": 1
}
```

Chaincode部署请求的响应内容中，包含了一个确认请求已经成功完成的`status`字段。成功的部署请求，其响应内容还包含部署时生成的chaincode hash，后续发给这个chaincode的调用和查询请求都需要带上这个hash。

Chaincode部署的响应: 

```
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        "message": "52b0d803fc395b5e34d8d4a7cd69fb6aa00099b8fabed83504ac1c5d61a425aca5b3ad3bf96643ea4fdaac132c417c37b00f88fa800de7ece387d008a76d3586"
    },
    "id": 1
}
```

调用一个chaincode时，用[ChaincodeSpec](https://github.com/hyperledger/fabric/blob/v0.6/protos/chaincode.proto#L60)来识别调用请求中的chaincode信息。注意，chaincode的`name`字段就是部署请求中返回的hash。

非安全模式的chaincode调用请求: 

```
{
  "jsonrpc": "2.0",
  "method": "invoke",
  "params": {
      "type": 1,
      "chaincodeID":{
          "name":"52b0d803fc395b5e34d8d4a7cd69fb6aa00099b8fabed83504ac1c5d61a425aca5b3ad3bf96643ea4fdaac132c417c37b00f88fa800de7ece387d008a76d3586"
      },
      "ctorMsg": {
         "args":["invoke", "a", "b", "100"]
      }
  },
  "id": 3
}
```

如果在安全模式下调用chaincode，就需要在上面的请求里添加`secureContext`元素，其值是已注册且登录用户的registrationID。

安全模式下的chaincode调用请求(添加了`secureContext`元素): 

```
{
  "jsonrpc": "2.0",
  "method": "invoke",
  "params": {
      "type": 1,
      "chaincodeID":{
          "name":"52b0d803fc395b5e34d8d4a7cd69fb6aa00099b8fabed83504ac1c5d61a425aca5b3ad3bf96643ea4fdaac132c417c37b00f88fa800de7ece387d008a76d3586"
      },
      "ctorMsg": {
         "args":["invoke", "a", "b", "100"]
      },
      "secureContext": "lukas"
  },
  "id": 3
}
```

Chaincode调用请求的响应内容中，包含了一个确认请求已经成功完成的`status`字段。成功的调用请求，其响应内容中还包含这次事务的事务id。事务的执行是异步的，客户端可以在事务被提交到系统之后，用事务id检查事务的状态。

Chaincode调用的响应: 

```
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        "message": "5a4540e5-902b-422d-a6ab-e70ab36a2e6d"
    },
    "id": 3
}
```

查询一个chaincode时，用[ChaincodeSpec](https://github.com/hyperledger/fabric/blob/v0.6/protos/chaincode.proto#L60)来识别查询请求中的chaincode信息。注意，chaincode的`name`字段就是部署请求中返回的hash。

非安全模式的chaincode查询请求: 

```
{
  "jsonrpc": "2.0",
  "method": "query",
  "params": {
      "type": 1,
      "chaincodeID":{
          "name":"52b0d803fc395b5e34d8d4a7cd69fb6aa00099b8fabed83504ac1c5d61a425aca5b3ad3bf96643ea4fdaac132c417c37b00f88fa800de7ece387d008a76d3586"
      },
      "ctorMsg": {
         "args":["query", "a"]
      }
  },
  "id": 5
}
```

如果在安全模式下查询chaincode，就需要在上面的请求里添加`secureContext`元素，其值是已注册且登录用户的registrationID。

安全模式下的chaincode查询请求(添加了`secureContext`元素): 

```
{
  "jsonrpc": "2.0",
  "method": "query",
  "params": {
      "type": 1,
      "chaincodeID":{
          "name":"52b0d803fc395b5e34d8d4a7cd69fb6aa00099b8fabed83504ac1c5d61a425aca5b3ad3bf96643ea4fdaac132c417c37b00f88fa800de7ece387d008a76d3586"
      },
      "ctorMsg": {
         "args":["query", "a"]
      },
      "secureContext": "lukas"
  },
  "id": 5
}
```

Chaincode查询请求的响应内容中，包含了一个确认请求已经成功完成的`status`字段。成功的调用请求，其响应内容中还包含对应的`message`，`message`的内容是chaincode定义的，其值取决于chaincode的实现，可以是一个字符串，也可以是一个数字。

Chaincode查询的响应: 


```
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        "message": "-400"
    },
    "id": 5
}
```

#### 网络

* **GET /network/peers**

使用网络API可以获取组成区块链网络的节点网络信息。

REST服务/network/peers会返回目标节点上所有正存在的网络连接信息。包含validating peer和non-validating peer。peer的信息以[`PeersMessage`](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L138)类型返回，`PeersMessage`中包含一个[`PeerEndpoint`](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L127)数组。

```
message PeersMessage {
    repeated PeerEndpoint peers = 1;
}
```

```
message PeerEndpoint {
    PeerID ID = 1;
    string address = 2;
    enum Type {
      UNDEFINED = 0;
      VALIDATOR = 1;
      NON_VALIDATOR = 2;
    }
    Type type = 3;
    bytes pkiID = 4;
}
```

```
message PeerID {
    string name = 1;
}
```

#### 注册

* **POST /registrar**
* **DELETE /registrar/{enrollmentID}**
* **GET /registrar/{enrollmentID}**
* **GET /registrar/{enrollmentID}/ecert**
* **GET /registrar/{enrollmentID}/tcert**

使用注册API管理终端用户在CA上的注册。这些API用于在CA上注册一个用户，检查一个用户是否已经注册了，删除一个用户的登录令牌以阻止该用户后续发起事务。注册API还可以从系统中取得注册和事务证书。

/registrar服务用于在CA中注册用户。[devops.proto](https://github.com/hyperledger/fabric/blob/v0.6/protos/devops.proto#L50)定义了注册时需要提交的保密内容。

```
message Secret {
    string enrollId = 1;
    string enrollSecret = 2;
}
```

注册请求的响应是一个注册成功信息或包含失败原因的错误信息。下面展示的是个用于注册用户'lukas'的有效保密信息的例子。

```
{
  "enrollId": "lukas",
  "enrollSecret": "NPKYL39uKbkj"
}
```

/registrar/{enrollmentID}服务用于验证一个给定的用户是否已在CA中注册过。如果是，会返回一个确认信息，否则返回一个授权错误。

DELETE /registrar/{enrollmentID}服务用于删除指定用户的登录令牌。删除成功会返回一个确认信息，否则返回一个授权错误。这个服务不需要请求内容。注意，一个registrationID和registrationPW只能在CA上注册一次，如果这个registrationID对应的用户信息被删除了，这个registrationID也不能用于第二次注册。

GET /registrar/{enrollmentID}/ecert服务用于从CA的本地存储中获取指定用户的注册证书。如果该用户已经在CA上注册了，会响应被URL编码后的注册证书，否则返回错误。客户需要先进行URL解码才能使用该证书，URL解码可以用"net/url"包下的QueryUnescape方法。

/registrar/{enrollmentID}/tcert用于给已在CA上注册的用户提供事务证书。如果用户已注册，会返回一个包含一组URL编码后的事务证书确认信息，否则返回错误。可以通过一个可选的请求查询参数'count'指定想要的事务证书的数量，不传默认返回一个证书，'count'不能大于500。客户需要先进行URL解码才能使用该证书，URL解码可以用"net/url"包下的QueryUnescape方法。

#### 事务

* **GET /transactions/{UUID}**

用/transactions/{UUID}服务能获取区块链中与该UUID匹配的事务的信息。[fabric.proto](https://github.com/hyperledger/fabric/blob/v0.6/protos/fabric.proto#L28)中定义里返回的事务信息。

```
message Transaction {
    enum Type {
        UNDEFINED = 0;
        CHAINCODE_DEPLOY = 1;
        CHAINCODE_INVOKE = 2;
        CHAINCODE_QUERY = 3;
        CHAINCODE_TERMINATE = 4;
    }
    Type type = 1;
    bytes chaincodeID = 2;
    bytes payload = 3;
    string uuid = 4;
    google.protobuf.Timestamp timestamp = 5;

    ConfidentialityLevel confidentialityLevel = 6;
    bytes nonce = 7;

    bytes cert = 8;
    bytes signature = 9;
}
```

可以看[协议规范](https://github.com/hyperledger/fabric/blob/v0.6/docs/protocol-spec.md)6.2部分的REST API，有更多的REST信息和更详细的例子。

### 设置Swagger-UI

[Swagger](http://swagger.io/)是一个方便的方案，可以让你用一个文件描述，记录REST API。fabric的REST API被描述在[rest_api.json](https://github.com/hyperledger/fabric/blob/v0.6/core/rest/rest_api.json)中。直接使用Swagger-UI和peer节点交互需要把可用的Swagger定义文件上传到[Swagger服务](http://swagger.io/)。或者，也可以通过下面的说明在本地机器上安装Swagger。

1. 可以使用Node.js提供rest_api.json服务。这样做要先在本地安装Node.js，如果还未安装，可以下载[Node.js](https://nodejs.org/en/download/)包并安装。

2. 安装Node.js的http-server包: 

    `npm install http-server -g`

3. 启动http-server，对外提供rest_api.json: 

    ```
    cd /opt/gopath/src/github.com/hyperledger/fabric/core/rest
    http-server -a 0.0.0.0 -p 5554 --cors
    ```

4. 确保能用这个地址在浏览器中访问API的描述文档: 

    `http://localhost:5554/rest_api.json`

5. 下载Swagger-UI包: 

    `git clone https://github.com/swagger-api/swagger-ui.git`

6. 进入/swagger-ui/dist目录，点击index.html文件在浏览器中打开Swagger-UI界面。

7. 启动一个peer节点，不连接到一个主或验证节点下: 

    ```
    cd /opt/gopath/src/github.com/hyperledger/fabric
    build/bin/peer node start
    ```

8. 如果你需要在这个本地peer节点上构建一个测试区块链，可以运行TestServerOpenchain_API_GetBlockCount测试来构建一个测试区块链，然后重启peer进程。这个测试实现在[api_test.go](https://github.com/hyperledger/fabric/blob/v0.6/core/rest/api_test.go)，它将创建一个5个区块的区块链。

    ```
    cd /opt/gopath/src/github.com/hyperledger/fabric/core/rest
    go test -v -run TestServerOpenchain_API_GetBlockCount
    ```

9. 回到浏览器中的Swagger-UI界面，加载API描述文档。现在你应该可以直接从Swagger中调用查询请求到预构建的区块链中了。

## Node.js应用程序

可以通过Node.js应用程序和peer进程交互。一种方式是基于Swagger API描述文档，[rest_api.json](https://github.com/hyperledger/fabric/blob/v0.6/core/rest/rest_api.json)和[swagger-js插件](https://github.com/swagger-api/swagger-js)。另一种方式是依赖IBM Blockchain [JS SDK](https://github.com/IBM-Blockchain/ibm-blockchain-js)。

### [使用Swagger JS插件](https://github.com/hyperledger/fabric/blob/v0.6/docs/API/Samples/Sample_1.js)

* 示范从Node.js应用程序和peer节点的交互
* 使用Node.js swagger-js插件: https://github.com/swagger-api/swagger-js

**运行:**

1. 构建安装[fabric core](https://github.com/hyperledger/fabric/blob/v0.6/docs/dev-setup/build.md)

    ```
    cd /opt/gopath/src/github.com/hyperledger/fabric
    make peer
    ```

2. 只运行一个本地peer节点(不是完整的网络): 

    `build/bin/peer node start`

3. 安装测试区块链的数据结构(只有5个区块)，在Vagrant里面运行一个测试，然后重启peer: 

    ```
    cd /opt/gopath/src/github.com/hyperledger/fabric/core/rest
    go test -v -run TestServerOpenchain_API_GetBlockCount
    ```

4. 在本机上启动http-server，对外提供rest_api.json: 

    ```
    npm install http-server -g
    cd /opt/gopath/src/github.com/hyperledger/fabric/core/rest
    http-server -a 0.0.0.0 -p 5554 --cors
    ```

5. 下载并解压[Sample_1.zip](https://github.com/hyperledger/fabric/blob/v0.6/docs/API/Samples/Sample_1.zip)

    ```
    unzip Sample_1.zip -d Sample_1
    cd Sample_1
    ```

6. 修改[openchain.js](https://github.com/hyperledger/fabric/blob/v0.6/docs/API/Samples/Sample_1.js)，把api_url修改成正确的值: 

    `var api_url = 'http://localhost:5554/rest_api.json';`

7. 运行Node.js应用程序

    `node ./openchain.js`

控制台上会有一些输出，程序在最后会挂起一会。这是对的，因为程序要等待调用事务执行完成之后再执行查询。可以在Sample_1目录下的'openchain_test'文件里看到一份样本输出。

### [弹珠示例应用](https://github.com/IBM-Blockchain/marbles)

* 另一个通过Node.js应用和peer节点进行交互的示例
* 以IBM Bluemix服务的形式部署一个区块链应用的示例

别惊讶，这个应用程序要演示通过IBM区块链在用户之间转移弹珠。我们要用Node.js和一点GoLang来做，这个应用的后台代码是运行在我们区块链网络中的GoLang代码。Chaincode自己会创建一个弹珠，存到chaincode的状态中。Chaincode自己能在安装的时候以键值对存储字符串数据。我们将把JSON对象字符串化来存储更复杂的数据。

更多关于IBM区块链弹珠示例的信息，安装，介绍，请访问[这个页面](https://github.com/IBM-Blockchain/marbles)。

### [商业票据示例应用程序](https://github.com/IBM-Blockchain/cp-web)

* 另一个通过Node.js应用和peer节点进行交互的示例
* 以IBM Bluemix服务的形式部署一个区块链应用的示例

这是一个IBM区块链怎么实现一个商业票据交易网络应用的示例。示例有几部分: 

* 一个网络上用于创建新用户的接口
* 一个为交易创建商业票据的接口
* 一个用于现有交易买卖的交易中心
* 一个只为审计者开放的用于检查交易信息的特殊接口

更多关于IBM区块链商业票据示例的信息，安装，介绍，请访问[这个页面](https://github.com/IBM-Blockchain/cp-web)。
