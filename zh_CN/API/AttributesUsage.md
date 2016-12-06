# Attributes usage

## 概述

属性功能允许chaincode在事务认证中使用扩展数据。这些属性由属性证书颁发机构（ACA）认证，因此链码可以信任属性值的真实性。要查看关于属性设计的完整文档，请阅读“[属性支持](../tech/attributes.md)”。

属性功能允许chaincode使用事务证书中的扩展数据。这些属性由属性证书颁发机构（ACA）认证，因此链码可以信任属性值的真实性。

要查看关于属性设计的完整文档，请阅读“属性支持”。

## 用例：有效计数器

属性功能的一个常见用例是基于属性的访问控制（ABAC），它允许根据调用者证书中携带的属性值将特定权限授予链码调用者。

“[有效计数器](../../examples/chaincode/go/authorizable_counter/authorizable_counter.go)”是ABAC的一个简单示例，在这种情况下，只有“position”属性具有值“软件工程师”的调用者才能够增加计数器。另一方面，任何调用者都能够读取计数器值。

为了实现这个例子，我们使用'[VerifyAttribyte](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.VerifyAttribute) '函数从chaincode代码中检查属性值。

```
isOk, _ := stub.VerifyAttribute("position", []byte("Software Engineer")) 
//这里调用ABAC API来验证属性，只要该值被验证，计数器将增加。
if isOk {
    //递增计数器代码
}
```

通过使用“[属性支持](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim/crypto/attr)”API可以实现相同的行为，在这种情况下必须实例化属性处理程序。

```
attributesHandler, _ := attr.NewAttributesHandlerImpl(stub)
isOk, _ := attributesHandler.VerifyAttribute("position", []byte("Software Engineer"))
if isOk {
    //递增计数器代码
}
```

如果属性被多次访问，则使用`attributeHandler`更有效，因为处理程序使用高速缓存来存储值和键。

为了获取属性值，如果只是验证它，可以使用以下代码：

```
attributesHandler, _ := attr.NewAttributesHandlerImpl(stub)
value, _ := attributesHandler.GetValue("position")
```

## 开启属性

要使用此功能，必须在membersrvc.yaml文件中设置以下属性：

- aca.enabled = true

另一种方法是使用环境变量：

```
MEMBERSRVC_CA_ACA_ENABLED=true ./membersrvc
```

## 开启属性加密*

为了使用属性加密，必须在membersrvc.yaml文件中设置以下属性：

- tca.attribute-encryption.enabled = true

或者属性环境变量:

```
MEMBERSRVC_CA_ACA_ENABLED=true MEMBERSRVC_CA_TCA_ATTRIBUTE-ENCRYPTION_ENABLED=true ./membersrvc
```

### 部署API使用属性

#### CLI

```
$ ./peer chaincode deploy --help
将指定的chaincode部署到网络。

Usage:
  Peer 链码部署

Global Flags:
  -a, --attributes="[]": JSON格式的chaincode的用户属性
  -c, --ctor="{}": JSON格式的chaincode的构造函数消息
  -l, --lang="golang": 语言编写链码
      --logging-level="": 默认日志记录级别和覆盖，有关完整语法，请参阅core.yaml
  -n, --name="": 部署事务返回的chaincode的名称
  -p, --path="": chaincode的路径
      --test.coverprofile="coverage.cov": Done
  -t, --tid="": 自定义ID生成算法的名称（hash和解码） e.g. sha256base64
  -u, --username="": 启用安全性时，chaincode操作的用户名
  -v, --version[=false]: 显示Fabric peer服务器的当前版本
```

要部署具有属性“company”和“position”的chaincode，它应该以以下方式编写：

```
./peer chaincode deploy -u userName -n mycc -c '{"Function":"init", "Args": []}' -a '["position", "company"]'
```

#### REST

```
POST host:port/chaincode

{
  "jsonrpc": "2.0",
  "method": "deploy",
  "params": {
    "type": 1,
    "chaincodeID":{
        "name": "mycc"
    },
    "ctorMsg": {
        "function":"init",
        "args":[]
    }
    "attributes": ["position", "company"]
  },
  "id": 1
}
```

### 调用使用属性的API

#### CLI

```
$ ./peer chaincode invoke --help
调用指定的chaincode。

Usage:
  peer chaincode 调用 [flags]

Global Flags:
  -a, --attributes="[]": JSON格式的chaincode的用户属性
  -c, --ctor="{}": JSON格式的chaincode的构造方法消息
  -l, --lang="golang": 写入chaincode的语言
      --logging-level="": 默认日志记录级别和覆盖，请参阅core.yaml获取完整语法
  -n, --name="": 部署事务返回的chaincode的名称
  -p, --path="": chaincode的路径
      --test.coverprofile="coverage.cov": Done
  -t, --tid="": 自定义ID生成算法（hash和解码）的名称。 e.g. sha256base64
  -u, --username="": 启用安全性时chaincode操作的用户名
  -v, --version[=false]: 显示Fabric peer服务器的当前版本
```

要调用具有属性“company”和“position”的“有效计数器”，它应该写成如下：

```
./peer chaincode invoke -u userName -n mycc -c '{"Function":"increment", "Args": []}' -a '["position", "company"]'
```

#### REST

```
POST host:port/chaincode

{
  "jsonrpc": "2.0",
  "method": "invoke",
  "params": {
    "type": 1,
    "chaincodeID":{
        "name": "mycc"
    },
    "ctorMsg": {
        "function":"increment",
        "args":[]
    }
    "attributes": ["position", "company"]
  },
  "id": 1
}
```

### 查询API利用属性

#### CLI

```
$ ./peer chaincode query --help
使用指定的chaincode进行查询

Usage:
  peer chaincode 查询 [flags]

Flags:
  -x, --hex[=false]: 如果为true，则以十六进制输出查询值字节数组。与--raw不兼容
  -r, --raw[=false]: 如果为true，则将查询值作为原始字节输出，否则格式为可打印字符串


Global Flags:
  -a, --attributes="[]": JSON格式的chaincode的用户属性
  -c, --ctor="{}": JSON格式的chaincode的构造方法消息
  -l, --lang="golang": 写入chaincode的语言
      --logging-level="": 默认日志记录级别和覆盖，请参阅core.yaml获取完整语法
  -n, --name="": 部署事务返回的chaincode的名称
  -p, --path="": chaincode的路径
      --test.coverprofile="coverage.cov": Done
  -t, --tid="": 自定义ID生成算法（hash和解码）的名称。 e.g. sha256base64
  -u, --username="": 启用安全性时chaincode操作的用户名
  -v, --version[=false]: 显示Fabric peer服务器的当前版本
```

要用属性“company”和“position”查询“可自动计数器”，应该这样写：

```
./peer chaincode query -u userName -n mycc -c '{"Function":"read", "Args": []}' -a '["position", "company"]'
```

#### REST

```
POST host:port/chaincode

{
  "jsonrpc": "2.0",
  "method": "query",
  "params": {
    "type": 1,
    "chaincodeID":{
        "name": "mycc"
    },
    "ctorMsg": {
        "function":"read",
        "args":[]
    }
    "attributes": ["position", "company"]
  },
  "id": 1
}
```

- **属性加密还不可用**。
