## oAuth

`Spring Security`是一个框架，可以拦截Http请求，实现了oAuth2的标准。

是认证授权与资源保护的解决方案。通过Session共享也是单点登录的解决方案。授权时效性的解决方案。

oAuth2是一个协议、标准，即提供很多接口，Spring Security实现了这些接口，这就是Spring Security oAuth2框架。当然还有Shiro oAuth2框架。

#### 角色

+ 第三方应用程序（客户端）

+ 资源所有者（用户）

+ HTTP 服务提供者

  + 认证服务器

  + 资源服务器

### oAuth2开放平台

以登录CSDN的QQ验证为例

1. 首先点击CSDN上的QQ第三方认证图标，打开QQ的登录页面。

2. 输入QQ的用户名和密码，点击`授权并登录`，这时是将用户名和密码发送给QQ认证服务器，

   如果用户名密码匹配，认证服务器会返回给客户端一个`Token`，客户端再携带该`Token`去请求访问QQ认证服务器，获得QQ服务器的资源，比如昵称、头像等。

3. CSDN客户端获得QQ资源后，创建该QQ的用户，并完成登录，可以访问CSDN的资源。

### Access Token

Access Token 是客户端访问资源服务器的令牌。拥有这个令牌代表着得到用户的授权。为了安全考虑，该Token是有一定期限的，如果Token过期，用户需要重新登录，获取新的Token，这样就比较麻烦，于是 oAuth2.0 引入了 Refresh Token 机制。

### Refresh Token

Refresh Token 的作用是用来刷新 Access Token。

Refresh Token **一定是保存在客户端的服务器上** ，为了防止泄露。不会暴露给客户端。

oAuth2.0 引入了 `client_secret` 机制。即每一个 `client_id` 都对应一个 `client_secret`。这个 `client_secret` 会在客户端申请 `client_id` 时，随 `client_id` 一起分配给客户端。**客户端必须把 `client_secret` 妥善保管在服务器上**，决不能泄露。刷新 Access Token 时，需要验证这个 `client_secret`。

### 客户端授权模式

+ 简化模式 

  没有自己的后台，只有客户端

+ 授权码模式

  最安全，也是官方推荐的模式。有自己的后台、资源服务器、认证服务器。

  认证服务器将授权码返回给客户端，客户端使用该授权码从认证服务器取得Token，再使用该Token访问资源服务器。客户端只能使用一次授权码，之后就失效了。

+ 密码模式

  两家企业关系比较信任，或同一家企业有不同的产品线，有定制化的授权页面

+ 客户端模式

  关系更进一步，同一产品线的不同服务，没有登录页面
