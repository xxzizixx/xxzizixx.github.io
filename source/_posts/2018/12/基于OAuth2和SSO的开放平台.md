---
title: 基于OAuth2的开放平台
date: 2018-12-13 18:51:19
tags: 
---

<br>
<br>

# 前序
最近部门搭建了一个`基于OAuth2和SSO的统一用户认证开放平台`，因为是搭建初期，文档和平台建设还不够完善，需要对接的第三方又比较多，所以大伙都在忙着充当一个`“开放平台客服”`的角色在忙着辅助第三方接入平台的事情。
那么，既然是辅助别人接入，首先我们自己就得搞清楚一下三个问题：
* `什么是开放平台?`
* `什么是OAuth2?`
* `什么是SSO？`
带着这三个问题的思考，进行下面的阅读。

<br>
<br>


# 开放平台介绍
### **什么是开放平台？**

`百度百科定义`：
> 开放平台(Open Platform)在软件行业和网络中，开放平台是指软件系统通过公开其应用程序编程接口(API)或函数(function)来使外部的程序可以增加该软件系统的功能或使用该软件系统的资源，而不需要更改该软件系统的源代码。 

`通俗应景的理解`：
> 开放平台就是互联网企业，将自己内部的资源(一般是数据，比如用户数据，平台业务数据等)，以技术的手段(一般是RESTFul接口API)，开放给受控的第三方合作伙伴(一般是指在平台注册登记并通过平台审核的用户)，或公司内部的其它一些产品(指平台所属公司开发的其他产品或者我们常说的全家桶)，形成一个安全受控的资源暴露平台(受控指接入的第三方都是通过平台认证许可的并在平台的监管下，暴露指共享资源以API接口的方式提供给第三方)。

<br>


### **为什么要搭建开放平台**

搭建开放平台的意义，一般在于：
* **1.搭建基于API的生态体系**
* **2.利用开放平台，搭建基于计费的API数据平台**
* **3.为APP端提供统一接口管控平台，类似于网关的概念**
* **4.为第三方合作伙伴的业务对接提供授信可控的技术对接平台**

<!--简单通俗点的说法是：-->
<!--* **1.我这里有你需要的东西，(比如用户数据<如新浪微博用户数据>、或专利技术<如环信即时通讯>)，想用吧？可以，待我搞个开放平台来监管你并根据你的日活流量来收费；**-->
<!--* **2.需要接入太多不同第三方的服务，每个第三方之间编码各有差异导致接入的规范不同，需要耗费大量的人力去一个个配合接入，并且接入大量三方后差异化太大导致项目工程代码可读性和维护性太差、臃肿不堪，  与其一个一个的配合别人接入，倒不如自己定制一个接入规范**-->
<!--* **3.**-->
<!--* **4.**-->

<br>




### **开放平台体系结构图**

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fy59ouca7hj31ar0lgagf.jpg)
<center> [开放平台体系结构思维导图](https://ws2.sinaimg.cn/large/006tNbRwgy1fy59gqo7p3j30ne0gj0ug.jpg) </center>

<br>

这里我做了一个思维导图，里面包含了开放平台的四个核心模块以及每个模块的作用：
##### **平台门户**
平台门户负责向第三方展示用于进行业务及技术集成的管理界面，至少包含以下几个功能：
* 服务商入住(第三方合作伙伴入住)
* 应用配置(第三方应用管理)
* 权限申请(一般包括接口权限和字段权限)
* 运维中心(开放平台当前服务器、接口状态，服务商接口告警等)
* 帮助中心(入住流程说明，快速接入说明，API文档等)

##### **鉴权服务**
鉴权服务负责整个平台的安全性
* 接口调用鉴权(第三方合作伙伴是否有权限调用某接口)
* 用户授权管理(用户对某个第三方应用获取改用户信息的权限管理)
* 用户鉴权(平台用户的鉴权)
* 应用鉴权(第三方合作伙伴的应用是否有权调用该平台)

##### **开放接口**
开放接口用于将平台数据暴露给合作伙伴
* 平台用户接口(用于获取公司APP生态链中的用户信息)
* 平台数据接口(平台中的一些开放数据)
* 其它业务接口(平台开放的一些业务数据)

##### **运营系统**
运营系统是整个平台的后台业务管理系统，负责对第三方合作伙伴提出的各种申请进行审核操作，对当前应用的操作进行审计工作，对当前业务健康度进行监控等
* 服务商管理(对第三方合作伙伴的资质进行审核、操作)
* 应用管理(对第三方应用进行审核、上下线管理)
* 权限管理（对合作伙伴申请的资源进行审核、操作）
* 统计分析(监控平台当前运行状态，统计平台业务数据)

<br>
<br>




# OAuth2介绍

### **什么是OAuth2？**

`百度百科定义`：
* **OAuth**
> OAuth协议为用户资源的授权提供了一个安全的、开放而又简易的标准。 与以往的授权方式不同该授权不会使第三方触及到用户的账户信息，即第三方无需使用用户名密码就可以申请该用户资源的授权，因此OAuth是安全的。OAuth是`Open Authorization`的简写。现在OAuth已经到2.0版本了。

* **OAuth2**
> OAuth2.0是OAuth的下一个版本，同时它也是授权领域的行业标准协议，OAuth2.0不支持向后兼容，即不支持OAuth1.0，彻底废止了OAuth1.0协议，OAuth2.0致力于使客户端开发者通过更简单的流程为Web应用、桌面应用以及手机客户端等设备进行授权。2012年10月，OAuth2.0协议正式发布为RFC6749。目前各大开放平台，如新浪、腾讯、豆瓣、人人、网易等等也都是以OAuth2.0协议作为支撑。

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fy5d3y59i6j30j00bh77a.jpg)

`通俗应景的理解`：
> OAuth2协议，定义了一套用户、第三方服务和存储着用户数据的平台之间的交互规则，可以使得用户无需将自己的用户名和密码暴露给第三方，即可使第三方应用获取用户在该平台上的数据，最常见的场景便是现在互联网上的各种使用XXX账号登录。

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fy63ef7or1j30m60c4myh.jpg)

<br>

### **OAuth2协议角色图**
<br>

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fy5eq9vq8uj31f40mq4qp.jpg)


<br>

OAuth2协议的四个角色，我也做了一个角色的思维导图，包括各角色的作用，其中的6个认证流程是：
* 1、第三方应用向用户请求授权，希望获取用户数据  
应用场景：用户打开客户端以后，客户端要求向资源所有者（即用户）给予授权；
* 2、用户同意授权
应用场景：用户同意给予客户端授权
* 3、第三方应用拿着用户授权，向平台索要用户access token
应用场景：客户端使用上一步获得的授权，向认证服务器申请令牌。
* 4、平台校验第三应用合法性及用户授权真实性后，向平台发放用户access token
应用场景：认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
* 5、第三方应用拿着用户access token向平台索要用户数据
应用场景：客户端使用令牌，向资源服务器申请获取资源。
* 6、平台在校验用户access token真实性后，返回用户数据
应用场景：资源服务器确认令牌无误，同意向客户端开放资源。

#### 认证流程图也可以用下面的图来表示：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fy61r37hy1j30hs0d376b.jpg)

<br>



### **OAuth 2.0授权模式**
授权许可：客户用来获取访问令牌的资源所有者授权的凭证。
客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0协议规定了4种授权方式：

* **`授权码模式（authorization code）`**
* **`简化模式（implicit）`**
* **`密码模式（resource owner password credentials）`**
* **`客户端模式（client credentials）`**



#### **授权码模式（authorization code）**
授权代码授权类型用于获取访问令牌和刷新令牌，并针对机密客户端进行优化。它是一个基于重定向的流程，因此客户端必须能够与资源所有者的用户代理（通常是Web浏览器）并且能够从授权服务器接收传入请求（通过重定向）。授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。授权流程如下：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fy636kx3gtj30ha0cw765.jpg)

流程解析：
* (A)客户端通过用户代理，重定向请求授权服务器，所需参数：客户端标识及重定向URL；
* (B)用户选择是否给予客户端授权；
* (C)授权服务器给予客户端一个认证授权码，并跳转到指定的URI；
* (D)客户端使用该授权码，重定向到授权服务器，获取令牌；
* (E)授权服务器校验该授权码以及重定向的URI后，向客户端发送令牌或者更新令牌。


步骤解析：
用户同意授权后，授权服务器并未直接将令牌发送给客户端，而是先向客户端发送了一个授权码，authorization code，然后再携带该授权码，再一次请求授权服务器，校验无误后，再发送令牌（access token），这一步是在后台自动完成的，这样也使OAuth授权更加安全。


在该授权流程中，我们所需的几个参数，官方文档中要求指定的参数如下：
```
response_type 必选项 表示的是要求指定的授权类型，此处必须设置为：code
client_id 必选项 客户端的唯一标识
redirect_uri 可选项 重定向的URI
scope 可选项 授权的管道
state 建议项 表示客户端状态，授权服务器会将该状态原值返回
```

看完该流程以及所需的参数后，我们结合实际情况，还是上面提到的例子，以CSDN使用QQ登录为例，看一下该过程：
(A步骤)点击QQ登录，跳转到QQ帐号安全登录界面，即腾讯的互联平台，我们可以直接拿到此时CSDN要请求的地址：
```
https://graph.qq.com/oauth2.0/show?which=Login&display=pc&
client_id=100270989&
response_type=code&
redirect_uri=https://passport.csdn.net/account/login?pcAuthType=qq&state=test
```
这样我们可以很清楚的看到在上面的URL中，发起了一个get请求，他的授权类型为code 授权码模式，以及client_id,redirect_uri,state等信息。
(B步骤)此时页面也跳转到了用户代理的页面，让用户去决定是否要同意授权 ：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fy63ef7or1j30m60c4myh.jpg)

(C步骤)当用户输入用户名密码后点击授权并登录，服务器会先验证你输入的是否正确，如果正确，页面就会跳转到redirect_uri的地址中，而在这个过程中发生的变化是这样的,我们继续查看这时的URL地址：
```
https://passport.csdn.net/account/login?oauth_provider=QQProvider&
code=D185F3ED93E4F1B3C5F557E6112C7A9B&
state=test
```
(D步骤)我们可以看到授权并登录后它重定向到了我们指定的地址，而且还给予了一个code授权码，另外state状态还是在请求登录时的状态，在下一个步骤中，是客户端向授权服务器申请令牌，包含以下参数：
```
grant_type 表示授权模式，此处固定为authorization_code，必选
code 表示上一步获取到的授权码，必选
redirect_uri 表示重定向URI，必选
client_id 表示客户端ID，必选
```
在使用腾讯开放平台时，它会要求有一个client_secrect，是对应你客户端ID的一个私钥，它也是在客户端注册时产生的，所以在使用QQ登录获取令牌时，也必须指定该选项：client_secret,具体的可以看腾讯的开发文档。那么实际上申请令牌的这个过程我们是看不到的，返回authorization code 后，我们会看到我们的客户端此时已经登录成功了。例如：
```
POST /token HTTP/1.1
Host: graph.qq.com/oauth2.0/token
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb&client_secret=fdhsjfdsj32jjfhjdk
```
(E步骤)若以上返回成功，那么在第5步骤中会返回如下信息：
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache
  {
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "example_parameter":"example_value"
  }
```
里面包含了我们所需要的令牌access token以及有效期等信息。实际上在我们实际应用过程中，我们不止是到这里就结束了，我们会使用access token令牌再去请求资源服务器，获取可以被调用的资源，以及一些其他的操作。

**`授权码模式（authorization code）`**是我们最常见也用的最多的一种。

<br>


#### **简化模式（implicit）**
简化模式用于获取访问令牌（但它不支持令牌的刷新），并对运行特定重定向URI的公共客户端进行优化，而这一些列操作通常会使用脚本语言在浏览器中完成，令牌对访问者是可见的，且客户端也不需要验证。具体流程如下：
![](https://ws3.sinaimg.cn/large/006tNbRwgy1fy64mvu6lfj30fd0fsmyg.jpg)


流程解析：
* (A)客户端携带客户端标识以及重定向URI到授权服务器；
* (B)用户确认是否要授权给客户端；
* (C)授权服务器得到许可后，跳转到指定的重定向地址，并将令牌也包含在了里面；
* (D)客户端不携带上次获取到的包含令牌的片段，去请求资源服务器；
* (E)资源服务器会向浏览器返回一个脚本；
* (F)浏览器会根据上一步返回的脚本，去提取在C步骤中获取到的令牌；
* (G)浏览器将令牌推送给客户端。

步骤解析：
(A步骤)中客户端发出的HTTP请求，需要用到的参数，注意在这里要使用"application/x-www-form-urlencoded"格式：
```
* response_type 必选项，此值必须为"token"
* client_id 必选项
* redirect_uri 可选项
* scope 可选项
* state 建议选项
```
例如：
```
GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
    &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
  Host: server.example.com
```

(C步骤)中认证服务器回应客户端的URI，返回的参数包含：
```
* access_token 必选项
* token_type 必选项
* expires_in 建议选项
* scope 可选项
* state 必选项
```
例如：
```
HTTP/1.1 302 Found
    Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
        &state=xyz&token_type=example&expires_in=3600
```

<br>



#### **密码模式（resource owner password credentials）**
密码模式适合建立在客户端与资源所有者具有信任关系的情况下，例如它是一个设备的操作系统或者具有很高权限的应用。这种模式用户要向客户端提供自己的用户名密码，从而达到向服务提供商索取授权。授权服务器在启动此类型时，要特别小心，只有在其他的授权方式不被允许的情况下才可以使用这种授权模式。流程如下：
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fy65y5jncwj30fd091js3.jpg)

流程解析：
* (A)客户端要求使用资源所有者的密码；
* (B)资源所有者给予用户名密码后，客户端向授权服务器发起申请令牌的请求；
* (C)授权服务器将令牌发放给客户端。


步骤解析：
(B步骤)所需参数：
```
* grant_type 必选项 此值必须为"password"
* username 必选项 用户名
* password 必选项 密码
* scope 可选项
```
例如：
```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded
grant_type=password&username=johndoe&password=A3ddj3w
```
(C步骤)中认证服务器向客户端发送访问令牌：
```
HTTP/1.1 200 OK
    Content-Type: application/json;charset=UTF-8
    Cache-Control: no-store
    Pragma: no-cache

  {
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "example_parameter":"example_value"
  }
```

<br>




#### **客户端模式（client credentials）**
客户端模式是4种模式中最简单的一种模式。客户端可以使用客户端凭据请求访问令牌（或者其他支持的认证方式），在这种模式中，客户端占据主导地位，它不需要用户的同意，可以直接向授权服务器索取令牌，严格来说，该模式并不存在授权的问题，流程如下：
![](https://ws1.sinaimg.cn/large/006tNbRwgy1fy65y5jncwj30fd091js3.jpg)

流程解析：
* (A)客户端向认证服务器进行身份认证，并要求一个访问令牌。
* (B)认证服务器确认无误后，向客户端提供访问令牌。

步骤解析：
(A步骤)中客户端发出的HTTP请求，包含以下参数：
```
* grant_type 必选项 此值必须为client_crendentials
* scope 可选项
```
例如：
```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded
grant_type=client_credentials
```

(B步骤)授权服务器返回结果：
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache
  {
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "example_parameter":"example_value"
  }
```
<br>

<!--### **OAuth 2.0授权模式**-->

参考文档
[《理解OAuth 2.0授权》](https://www.cnblogs.com/Allen0910/p/8647935.html)
[《也说OAuth授权登录》](https://baijiahao.baidu.com/s?id=1595743642120820130&wfr=spider&for=pc)
[《搭建基于OAuth2和SSO的开放平台》](http://heartlifes.com)






