---
layout: post
title: Licode（二）：Nuve源码分析
category: 技术                                                                                                                                                
tags: [webRTC, Licode, Nuve]
keywords: webRTC,Licode,Nuve                                                                                                                                     
description: 
---

Licode的Nove组件旨在对服务端资源进行管理（CRUD），服务端资源包括：会议房间（videoconference rooms）、加入凭证（tokens）、参与用户（User）。Nuve组件的gitHub地址：<https://github.com/ging/licode/tree/master/nuve>，官方对Nuve的说明如下：

>Developers will manage rooms (creation and deletion) from their server apps.
>
> Typical server apps can create a room, and request access for their users to Nuve, via a token-based authentication mechanism.
> 
> This mechanism allows these servers to create access tokens, and they will provide these tokens to their clients.
>
> **Server talks to Nuve in order to do these actions.**

## 1、Nuve目录结构

* **nuveAPI**：Nuve App的实现代码
* **nuveClient**：NuveClient为Nuve App的客户端API
* **nuveClient_python**:nuveClient的python版本
* **nuveClient_ruby**:nuveClient的ruby版本
* **initNuve.sh**:Nuve App启动脚本
* **installNuve.sh**:NuveAPI依赖库安装、编译nuveClient脚本
* **log4js_configuration.json**:Nuve App Log配置文件

## 2、nuveAPI

### 2.1、总体架构
![image](/public/upload/img/2015-05-27-licode-nuve.md/nuve-api-architecture.png)

* **nuve.js：**通过调用resource系列类来完成CRUD操作，在进行CRUD操作之前，通过调用auth/nuveAuthenticator.js来完成权限验证，接口层
* **servicesResource.js/serviceResource.js：**
*  **roomsResource.js/roomResource.js：**会议房间资源，业务逻辑层
* **usersResource.js/userResource.js：**参与用户资源，业务逻辑层
* **tokenResource.js：**加入凭证资源，业务逻辑层
* **serviceRegistry.js、roomRegistry.js、tokenRegistry.js：** 通过调用dataBase.js完成各种资源的CRUL操作，数据逻辑层
* **dataBase.js：**mongodb部署配置相关，负责维护和mongodb服务的连接，和具体业务逻辑无关


```
Note:
- XXXsResource.js（复数形式）:负责创建、批量查询之类操作
- XXXResource.js（单数形式）:负责删除、更新、单个查询之类操作
```

### 2.2、nuve.js

nuve.js为NuveAPI的入口文件，Nuve App的表现层，以webservice的方式对外提供服务，urlpath和Resource的映射关系如下：[查看代码](https://github.com/ging/licode/blob/master/nuve/nuveAPI/nuve.js)

```
app.post('/rooms', roomsResource.createRoom);
app.get('/rooms', roomsResource.represent);

app.get('/rooms/:room', roomResource.represent);
app.delete('/rooms/:room', roomResource.deleteRoom);

app.post('/rooms/:room/tokens', tokensResource.create);

app.post('/services', servicesResource.create);
app.get('/services', servicesResource.represent);

app.get('/services/:service', serviceResource.represent);
app.delete('/services/:service', serviceResource.deleteService);

app.get('/rooms/:room/users', usersResource.getList);

app.get('/rooms/:room/users/:user', userResource.getUser);
app.delete('/rooms/:room/users/:user', userResource.deleteUser);
```

每次请求会先执行权限验证逻辑：[查看代码](https://github.com/ging/licode/blob/master/nuve/nuveAPI/nuve.js)

```
app.get('*', nuveAuthenticator.authenticate);
app.post('*', nuveAuthenticator.authenticate);
app.delete('*', nuveAuthenticator.authenticate);
```

nuveAuthenticator.authenticate的执行结果为：

 > This function has the logic needed for authenticate a nuve request. If the authentication success exports the service and the user and role (if needed). Else send back a response with an authentication request to the client.

nuve.js还提供了一个测试页面，在vagrant虚拟机中启动Nuve App后（Note：如何在vagrant中启动Nuve App，参见[Licode（一）：入门](2015-05-27-licode-nuve.html)的《mac X环境下，搭建licode测试环境》部分），在chrome浏览器地址栏里输入<http://localhost:3000/test>，可见一个如下界面：

![image](/public/upload/img/2015-05-27-licode-nuve.md/nuveapi-test.png)

其中，Service ID、Key配置在$ROOT/licode_config.js中：

```
  config.nuve.superserviceID = '555f70ee0c55fbbc84e5196f'; // default value: ''
  config.nuve.superserviceKey = '13518'; // default value: ''
```


### 2.3、cloudHandle.js

cloudHandle旨在调用erizoController App中的方法，从而在Nuve App中实现对erizoController App状态的干预，cloudHandle通过RabbitMQ来实现的RPC（Remote Procedure Call），[rpc.js](https://github.com/ging/licode/blob/master/nuve/nuveAPI/rpc/rpc.js)实现RPC的方式和[amqper.js](https://github.com/ging/licode/tree/master/erizo_controller/common/amqper.js)一样，可以认为amqper.js为rpc.js的升级版本，先看下cloudHandle的类图（Class Diagram）：

![image](/public/upload/img/2015-05-27-licode-nuve.md/cloud-hangle-class.png)

**rpc.js的原理**

Nuve App和erizoController App通过RabbitMQ维护了两队列，队列1用来传递调用消息，Nuve App发布调用消息后，会将callback保存至本地字典表中；erizoController App从队列1获取调用消息，完成方法调用之后，push调用结果消息至队列2中；Nuve App从队列2获取调用结果消息，从字典表中找到对应的callback并从字典表中删除，执行callback(调用消息)，至此一次RPC调用完成。从上面可以看出，这是个异步的过程，不会对Nuve App和erizoController App其他业务逻辑有任何影响。

### 2.4、nuveAuthenticator.js

nuveAuthenticator.js为Nuve App提供了权限验证功能，其实现原理：

NuveClient在发送http请求时构建Authorization头部，在Authorization中包含mauth_signature字段，mauth_signature值的计算方式如下：[查看代码](https://github.com/ging/licode/blob/master/nuve/nuveClient/src/N.API.js)

```
calculateSignature = function (toSign, key) {
    var hash, hex, signed;
    hash = CryptoJS.HmacSHA1(toSign, key);
    hex = hash.toString(CryptoJS.enc.Hex);
    signed = N.Base64.encodeBase64(hex);
    return signed;
};
```

Key为NuveClient事先和Nuve App约好的密钥，而toSign的值计算方式如下：[查看代码](https://github.com/ging/licode/blob/master/nuve/nuveClient/src/N.API.js)

```
var signed;
timestamp = new Date().getTime();
cnounce = Math.floor(Math.random() * 99999);
toSign = timestamp + ',' + cnounce;
toSign += ',' + username + ',' + role;
```

Nuve App收到NuveClient发送过来的请求后，从HttpHeader中提取Authorization，从Authorization中提取signed（calculateSignature的结果）、mauth_timestamp、mauth_cnonce、mauth_username、mauth_role，按照和NuveClient相同的方式计算出calculatedSignature，如果calculatedSignature === signed，验证通过，否则验证失败：[查看代码](https://github.com/ging/licode/blob/master/nuve/nuveAPI/auth/nuveAuthenticator.js)

```
var checkSignature = function (params, key) {
    "use strict";
    if (params.signature_method !== 'HMAC_SHA1') {
        return false;
    }
    var calculatedSignature = mauthParser.calculateClientSignature(params, key);
    if (calculatedSignature !== params.signature) {
        return false;
    } else {
        return true;
    }
};
```

### 2.5、xxxResource.js

**servicesResource.js/serviceResource.js类图（Class Diagram）如下：**

![image](/public/upload/img/2015-05-27-licode-nuve.md/service-resource-class.png)

**roomsResource.js/roomResource.js类图（Class Diagram）如下：**

![image](/public/upload/img/2015-05-27-licode-nuve.md/room-resource-class.png)

**usersResource.js/userResource.js类图（Class Diagram）如下：**

![image](/public/upload/img/2015-05-27-licode-nuve.md/user-resource-class.png)

**tokenResource.js类图（Class Diagram）如下：**

![image](/public/upload/img/2015-05-27-licode-nuve.md/token-resource-class.png)


### 2.6、xxxRegistry.js

xxxRegistry.js的功能是实现room、service、token的CURL，逻辑简单，所以这里不再一一赘述，直接参阅代码即可，这里仅给出类图（Class Diagram）：

![image](/public/upload/img/2015-05-27-licode-nuve.md/mdb-class.png)

room、service、token资源信息落地至mongodb中，mongodb配置在$ROOT/licode_config.js中：

```
config.nuve.dataBaseURL = "localhost/nuvedb"; // default value: 'localhost/nuvedb' 
```

Data base collections and its fields are:

```
room {name: '', [p2p: bool], [data: {}], _id: ObjectId}
service {name: '', key: '', rooms: Array[room], testRoom: room, testToken: token, _id: ObjectId}
token {host: '', userName: '', room: '', role: '', service: '', creationDate: Date(), [use: int], [p2p: bool], _id: ObjectId}
```

## 3、NuveClient

NuveClient本身为js实现，其为Video App提供调用接口，主要功能是发送HttpRequest请求至服务端，N.API.js类图（Class Diagram）如下：

![image](/public/upload/img/2015-05-27-licode-nuve.md/nuve-client-class.png)

## 4、写在最后

nuveClient_python和nuveClient_ruby功能上和nuveClient一样，不同语言版本而已，所以不再赘述

参考资料：

* licode官网文档：<http://lynckia.com/licode/architecture.html>
* AMQP协议：<http://langyu.iteye.com/blog/759663>