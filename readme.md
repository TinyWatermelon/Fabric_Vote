# 基于Hyperledger fabric 1.4.0的投票链码开发
#### vote chaincode based on hyperledger fabric 1.4.0
------

## 开发环境 envs
Ubuntu 18.04 LTS
![env](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/env1.PNG)
![env](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/env2.PNG)

## chaincode脚本 chaincode script
[点击此处查看 Click here](https://github.com/TinyWatermelon/Fabric_Vote/blob/master/vote.go)
三个功能，voteUser投票，getUserVoteById查看用户票数，getUserVote查看所有用户票数。
用户数据结构包含一个为string类用户名username 一个为int票数votenum
State DB同理，主键PK为username

## 具体步骤 lets begin

![vote.go](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/cdvote.PNG)
在目录文件内的chaincode/新建一个vote/文件，将vote.go文件拷贝至chaincode/vote/内

### 启动开发网络 start devmode

```shell
cd chaincode-docker-devmode/
docker-compose -f docker-compose-simple.yaml up
```
![docker0](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/docker0.PNG)

启动成功之后可用docker命令查看容器状态
```shell
docker ps 
```
![ensure](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/ensure0.PNG)
可以看到chaincode cli peer orderer 四个容器都在运行中
> * 如果之前没有正确释放cli容器会导致cli启动后报错退出，解决办法如图所示
![err0](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/err0.PNG)

### 编译脚本文件 build chaincode

进入链码节点容器
```shell
docker exec -it chaincode bash
```
然后编译链码，连接上peer
```shell
cd vote/
go build vote.go
CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./vote
```

> * 根据官方的文档，1.1.0之后的和peer之间通信用7052端口，很多旧版本的教程仍然是7051端口，结果就是会报错，而且错误提示会让你觉得是其他地方错了，这点很坑，请注意
> * [参考此处(stackoverflow.com)](https://stackoverflow.com/questions/48007519/unimplemented-desc-unknown-service-protos-chaincodesupport)
![err1](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/err1.PNG)

### 操作链码 run chaincode

打开一个新的控制行节点容器
```shell
docker exec -it cli bash
```
![docker2](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/docker2.PNG)

装载链码
```shell
peer chaincode install -p chaincodedev/chaincode/vote -n mycc -v 0
```
![install](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/install.PNG)
![installr](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/installresult.PNG)
成功后会返回200

初始化，根据vote.go定义无参留空
```shell
peer chaincode instantiate -n mycc -v 0 -c '{"Args":[]}' -C myc
```
![instantiater](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/instantiateresult.PNG)
成功后结果如上图所示

调用所写的函数,格式如下
```shell
peer chaincode invoke -n mycc -c '{"Args":["funcName"]}' -C myc //if there exists function parameter
                                                                //then args form will be 
                                                                //{"Args":["funcName","parameter"]}
                                                                //如果函数有参，格式如上
```
例如给用户投票
![voteUser](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/voteUser.PNG)
![voteUserr](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/voteUserresult.PNG)
成功后会返回200

多插入一些数据后可以查询总的票数
![getUserVote](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/getUser.PNG)
![getUserVoter](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/getUserresult.PNG)
结果如上图所示

也可以用username查询某个用户
![getUserbyid](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/getUserbyid.PNG)
![getUserbyid](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/getUserbyidresult.PNG)
结果如上图所示

### END
请务必养成释放容器的好习惯，能减少很多bug，真的
![end](https://github.com/TinyWatermelon/blog_pic/blob/master/hfv/end.PNG)
