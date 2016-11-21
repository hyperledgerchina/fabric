# Chaincode APIs

当chaincode函数`Init`, `Invoke` 或者 `Query` 被调用时,fabric将传递  
`stub *shim.ChaincodeStub`参数. `stub`可以通过调用API访问账本服务，事务的上下文，  
或者调用其他chaincode  

这些API被定义在[shim package](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim)中,用`godoc`生成. 它也包含 [chaincode.pb.go](https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/chaincode.pb.go) 中的函数，比如私有函数 `func (*Column) XXX_OneofFuncs`。 可以通过[chaincode.go](https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/chaincode.go) 查看函数定义，也通过例子[chaincode samples](https://github.com/hyperledger/fabric/tree/master/examples/chaincode)来熟悉如何使用这些函数.
