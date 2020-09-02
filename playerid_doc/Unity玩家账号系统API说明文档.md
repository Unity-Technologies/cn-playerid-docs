
- [Player ID](#playerid)
- [介绍](#introduction)
- [包](#package)
  * [API 约定](#apisc)
  * [认证 APIs](#aapis)
     * [POST /authentication/password](#pap)
     * [POST /authentication/external-token](#pae)
     * [POST /authentication/session-token](#pas1)
     * [POST /authentication/sign-up](#pas2)
     * [POST /authentication/anonymous](#paa)
     * [POST /authentication/link](#pal)
     * [POST /authentication/unlink](#pau)
     * [POST /authentication/create-credential](#pac3)
     * [POST /authentication/reset-password](#par)
     * [POST /authentication/reset-password/verify](#parv)
     * [POST /authentication/reset-password/confirm](#parc)
     * [POST /authentication/change-password](#pacp)
     * [POST /authentication/verify-email](#pav)
     * [POST /authentication/verify-email/confirm](#pavc)
   * [授权 APIs](#authoapis)
     * [GET /oauth2/auth](#goa)
     * [GET /oauth2/token](#got)
     * [GET /.well-known/openid-configuration](#gwo)
     * [GET /.well-known/jwks.json](#gwj)
   * [用户管理API](#manageapi)
     * [GET /users](#gu)
     * [GET /users/{id}](#gui)
     * [PUT /users/{id}](#pui)
     * [POST /users/{id}/disable](#puid)
     * [POST /users/{id}/enable](#puie)
     * [DELETE /users/{id}](#dui)
   * [核心数据模型](#datamodel)
     * [User](#user)
   * [Token 参考](#tokenr)
     * [Access Token](#accesstoken)
     * [ID Token](#idtoken)
     * [Refresh Token](#refreshtoken)
     * [Session Token](#sessiontoken)
  
***

  
# <a name="playerid"></a>Player ID
# <a name="introduction"></a>介绍
这些API旨在与Unity玩家账号系统（PlayerId）一起使用。

该服务使开发人员可以轻松安全地构建登录和玩家账号系统。

# <a name="package"></a>包
com.unity.playerid-cn包含了本文档中列出大多数API调用。

要开始使用下面列出的API，您需要通过Unity编辑器项目设置(Project Settings)创建ID 域(ID domain)和OAuth2客户端(OAuth2 client)。有关如何执行此操作的更多信息和步骤，请参阅[```《Unity玩家账号系统开发文档》```](../playerid_doc/Unity玩家账号系统开发文档.md)。

<font color=Red>```注意```</font>：此处的所有API都需要ID domain key作为Header的一部分。该key是从Unity Editor项目设置(Project Setting)的ID domain部分生成的字符串。对于授权API，您可能还需要添加Client ID到请求body中，该Client ID在Unity编辑器项目设置（Project Settings）的OAuth2 client生成。

| Key | Value | Notes |
| ------ | ------ | ------ |
|ID-Domain-Key | ID Domain 标识字符串 | 可以在Unity Editor 项目设置Project Settings ►  Player Identity ► Backends ► Unity UserAuth 中找到  |
| Client_Id | OAuth2 Client 标识字符串 |可以在 Unity Editor 的 Project Settings ► Player Identity ► Backends ► Unity UserAuth中找到|


## <a name="apisc"></a>API 约定
以下是UAS REST API的一些约定：

+ 大多数API请求和响应均为JSON格式。
+ <font color=Red>Base url 为```https://identity-api.unity.cn```</font>
+ 要在OAuth2 POST API中使用表单编码(form-encoded)的请求正文，请将header指定为：
 `Content-Type：application / x-www-form-urlencoded`
+ 在API响应中，如果字段是其类型的默认值，则该字段将被省略。例如，如果布尔字段为false，则该字段会从Response中省略，因为false是默认值。

下表显示类型的默认值。

| Type | Default | 
| ------ | ------ |
|boolean |false | 
| number | 0|
|string  |""|
|array   |empty array|

## <a name="aapis"></a>认证 APIs

使用**认证API**(Authentication APIs)来验证或更新用户。

身份**认证API**返回用户的信息和**ID令牌**(ID token)。

使用**ID令牌**(ID token)作为授权API(Authorization API)的输入，以获取**访问令牌**(access token)。

***

### <a name="pap"></a>POST /authentication/password

使用电子邮件/密码对用户进行身份验证。

**Authorization**

这是一个公开(public)API。不需要Authorization header。

**Request Body**

| **Field Path** | **Type** | 说明 |
| ------ | ------ | ------ |
|email |string | 用户登录的email  |
| password | string |用户账户的密码|

**Response Body**

| **Field Path** | **Type** | 说明 |
| ------ | ------ | ------ |
|userId |string | 认证用户的ID  |
| email | string |用户的邮箱|
|idToken|string|经过身份验证的用户的ID令牌(ID token)。该ID令牌(ID token)可以传递给授权服务器以完成身份验证流程。|
|sessionToken|string|会话令牌，用于为用户保留身份验证会话。它可能会改变；调用者应始终用响应中的值覆盖sessionToken。|
|expiresIn|string|ID令牌(ID token)过期时间。默认为3600秒。|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前provider。

**Error Responses**

| **Scenario** | **error code** | Example Response JSON |
| ------ | ------ | ------ |
|当电子邮件/密码为无效时，会有以下响应返回 |`INVALID_CREDENTIALS` | `{"error": "INVALID_CREDENTIALS","message": "Email/password is invalid."}`|
***
### <a name="pae"></a>POST /authentication/external-token
使用第三方令牌对用户进行身份验证。

**Authorization**

这是一个公开(public)API。不需要Authorization header。


**Request Body**

| **Field Path** | **Type** |**是否必须** |说明 |
| ------ | ------ | ------ | ------ |
|idProvider |string | Y  | ID provider的ID（例如：apple.com)
| idToken | string |N|ID provider 的ID令牌|
|accessToken|string|N|ID provider的访问令牌。|
|redirectUri|string|N|(仅OAuth）将ID provider重定向到用户提供的URL。|
|authCode|string|N|ID provider的验证码。|
|clientId|string|N|ID provider用来获取令牌的客户端ID。|

`NOTE:`

+ 每个ID provider通常只需要一种令牌。下表显示了ID provider及其必填字段：

| **ID Provider 类型** | **必要字段** |现在是否可用|
| ------ | ------ | ------ | 
|apple.com |idToken, clientId, redirectUri `OR` authCode, clientId, redirectUri | Y  |
|facebook.com|accessToken, redirectUri|N|
|google.com|idToken, redirectUri|N|


**Response Body**

| **Field Path** | **类型** |**说明**|
| ------ | ------ | ------ | 
|userId |string | 认证用户的ID |
|email|string|用户的email|
|idToken|string|经过身份验证的用户的ID令牌。该ID令牌可以传递给授权服务器完成身份验证流程。|
|sessionToken|string|会话令牌以保留用户的身份验证会话。它可能会改变；调用者应始终用response的值覆盖sessionToken。
|expiresIn|string|ID令牌(ID token)过期时间。默认为3600秒。|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前provider。|
|rawUserInfo|string|字符串化的JSON响应，其中包含所有对应的ID provider提供对应OAuth凭证数据。|

**Error Responses**

| **Scenario** | **error code** | Example Response JSON |
| ------ | ------ | ------ |
|当使用无效的第三方令牌 | `ID_PROVIDER_ERROR `| `{"error": "ID_PROVIDER_ERROR", "message": "invalid_grant"}`|

***

### <a name="pas1"></a>POST /authentication/session-token


使用会话令牌对用户进行身份验证。会话令牌作为身份验证响应的一部分返回。可以将其存储在应用程序/代理中以保留登录。

**Authorization**

这是一个公开(public)API。不需要Authorization header。

**Request Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|sessionToken |string| 认证的session token(会话令牌)|

**Response Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|userId |string| 认证用户的ID|
|idToken|string|经过身份验证的用户的ID令牌。该ID令牌可以传递给授权服务器完成身份验证流程。|
|email|string|用户的email|
|sessionToken|string|会话令牌以保留用户的身份验证会话。它可能会改变；调用者应始终用response的值覆盖sessionToken。|
|expiresIn|string|ID令牌(ID token)过期时间。默认为3600秒|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前provider。|


**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|当使用不合法的session token(会话令牌) |`INVALID_SESSION_TOKEN`| `{"error": "INVALID_SESSION_TOKEN","message": "Session token invalid."}`|
|在需要传递session token(会话令牌)地方没有传递|`MISSING_SESSION_TOKEN`|`{	"error": "MISSING_SESSION_TOKEN",	"message": "Session token is missing."}`
***
### <a name="pas2"></a>POST /authentication/sign-up
使用email/password 注册一个新用户

**Authorization**

这是一个公开(public)API。不需要Authorization header。

**Request Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|email |string| 用户新建的email|
|password|string|用户新建的密码|

**Response Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|userId |string| 认证用户的ID|
|email|string|新建用户的email|
|idToken|string|经过身份验证的用户的ID令牌。该ID令牌可以传递给授权服务器以完成身份验证流程。|
|sessionToken|string|会话令牌，用于为用户保留身份验证会话。它可能会改变；调用者应始终用response中的值覆盖sessionToken。|
|expiresIn|string|ID 令牌过期时间（ID Token）默认值为3600秒|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前provider。|

**Error Responses**


| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|当使用已注册过用户的电子邮件 |`ACCOUNT_EXISTS`| `{"error": "ACCOUNT_EXISTS","message": "Account already exists."`}|
|当用户输入密码小于6个字符|`WEAK_PASSWORD`|`{	"error": "WEAK_PASSWORD",	"message": "Password should be at least 6 characters"}`|
|当用户电子邮件不合法或者缺失|`INVALID_EMAIL`|`{	"error": "INVALID_EMAIL",	"message": "Email is invalid."}`|

***


### <a name="paa"></a>POST /authentication/anonymous
注册一个新匿名用户

**Authorization**

这是一个公开(public)API。不需要Authorization header。


**Request Body**

没有 request body.

**Response Body**


| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|userId |string| 认证用户的ID|
|idToken|string|经过身份验证的用户的ID令牌。该ID令牌可以传递给授权服务器以完成身份验证流程。|
|sessionToken|string|会话令牌，用于为用户保留身份验证会话。它可能会改变；调用者应始终用response中的值覆盖sessionToken。|
|expiresIn|string|ID 令牌过期时间（ID Token）默认值为3600秒|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前provider。|

***
### <a name="pal"></a>POST /authentication/link
将现有用户与外部ID提供者(ID provider)的身份相关联。

**Authorization**

需要用户access token才能调用此API。只有用户可以将外部ID提供者(ID providers)链接/取消链接到他们自己的帐户。传递访问令牌作为标准Authorization header。

| **key** | **value** | **描述** |
| ------ | ------ | ------ |
|Authorization |Bearer `<Token>`| 用户access token|



**Request Body**

| **Field Path** | **类型** | **required** |**描述**|
| ------ | ------ | ------ | ------ |
|idProvider |string| Y|ID 提供者(ID provider)的ID|
|idToken|string|N|来自外部ID 提供者(ID provider)的ID 令牌(ID token)|
|accessToken|string|N|来自外部ID 提供者(ID provider)的access token|
|redirectUri|string|N|（仅OAuth）IDP将用户重定向到的URI|
|authCode|string|N|来自外部ID 提供者(ID provider)的auth code |
|clientId|string|N|外部ID提供者(ID provider)用来获取令牌的client ID。|

```NOTE:```

每个ID提供者(ID provider)通常只需要一种令牌。下表显示了当前支持的ID提供者(ID provider)及其必填字段：


| **ID Provider 类型** | **必要字段** | 
| ------- | ------ |
|apple.com |idToken, clientId, redirectUri 或者 authCode, clientId, redirectUri| Y|ID 提供者(ID provider)的ID|
|idToken|accessToken, redirectUri|N|来自外部ID 提供者(ID provider)的ID 令牌(ID token)|
|google.com|idToken, redirectUri|N|来自外部ID 提供者(ID provider)的访access token|



**Response Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|userId |string| 认证用户的ID|
|email|string|用户的email|
|idToken|string|经过身份验证的用户的ID令牌。该ID令牌(ID token)可以传递给授权服务器以完成身份验证流程。|
|sessionToken|string|会话令牌(session token)，用于为用户保留身份验证会话。它可能会改变；调用者应始终用response中的值覆盖sessionToken。|
|expiresIn|string|ID 令牌(ID token)过期时间,默认值为3600秒|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前provider。|
|rawUserInfo|string|字符串化的JSON响应，其中包含所有对应的ID provider提供对应OAuth凭证数据。|


**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|尝试使用已链接到一个用户的社交账户 |`ACCOUNT_EXISTS`| `{	"error": "ACCOUNT_EXISTS","message": "The account is already linked to another user."}`|
|尝试登录一个目前不支持的ID provider的账号|`ID_PROVIDER_ERROR`|`{"error": "ID_PROVIDER_ERROR",	"message": "Invalid external account credential or ID provider type."}`|                                         
***
### <a name="pau"></a>POST /authentication/unlink
取消现有用户外部ID provider的身份链接。

**Authorization**

需要用户access token才能调用此API。只有用户可以将外部ID provider链接/取消链接到他们自己的帐户。传递access token作为标准Authorization header。

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|Authorization|Bearer `<Token>`| 用户的access token|


**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|idProviders|[]string| 需要取消链接IDs provider数组|

**Response Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|userId|string| 认证用户的ID|
|email|string|用户的email|
|user|[User](#user)|更新后的用户|


**Error Responses**


| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|用户尝试解除最后一个ID provider |`LAST_ID_PROVIDER`| `"{"error": "LAST_ID_PROVIDER", "message": "The account cannot unlink from its last ID provider."}`|                                              
|尝试解除提供密码的ID provider|`CANNOT_UNLINK_PASSWORD`|`{"error": "CANNOT_UNLINK_PASSWORD","message": "The account cannot unlink from the password provider."}`|    
                                
***
### <a name="pac3"></a>POST /authentication/create-credential
将凭据（用户名/密码）添加到现有用户（匿名用户或使用社交登录名登录的用户）。

**Authorization**
需要用户access token才能调用此API。只有用户可以将外部ID提供者(ID providers)链接/取消链接到他们自己的帐户。传递访问令牌作为标准Authorization header。


| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|Authorization|Bearer `<Token>`| 用户的access token|

**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|email|string| 用户的email|
|password|string|用户新建的password|

**Response Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|userId|string| 认证用户的ID|
|email|string|用户的email|
|user|[User](#user)|更新后的用户信息|


**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|尝试为已经有email/password 且已经在使用的用户创建凭据 email/password |`ACCOUNT_EXISTS`| `{	"error": "ACCOUNT_EXISTS","message": "Account already exists."}`|                                              
|用户创建的密码小于六个字符|`WEAK_PASSWORD`|`{"error": "WEAK_PASSWORD","message": "Password should be at least 6 characters"}`|                    
|email 缺失或者不合法|`INVALID_EMAIL`|`{"error": "INVALID_EMAIL","message": "Email is invalid."}`|
|用户已经创建email/password|`PASSWORD_AUTH_ALREADY_SETUP`|`{"error": "PASSWORD_AUTH_ALREADY_SETUP",	"message": "User already has password setup."}`|

***

### <a name="par"></a>POST /authentication/reset-password

触发向用户发送重置密码的电子邮件（使用用户的电子邮件地址）。

**Authorization**

这是一个公开(public)API。不需要Authorization header。

**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|email|string| 重置密码的用户email|
|locale|string|电子邮件区域设置。|



**Response**

没有 response body.

***

### <a name="parv"></a>POST /authentication/reset-password/verify

验证用于重置密码的验证码并返回电子邮件地址。

**Authorization**
这是一个公用API。不需要Authorization header。

**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|code|string| 重置邮件中的验证码|



**Response Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|email|string| 用于收取验证码的电子邮箱地址|

**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|发送没有验证码的请求 |`INVALID_PARAMETERS`| `{"error": "INVALID_PARAMETERS","message":"Code is missing."}`|                                                                                                                                

***

### <a name="parc"></a>POST /authentication/reset-password/confirm

确认重置密码(通过新输入的密码)。

**Authorization**

这是一个公开(public)API。不需要Authorization header。


**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|code|string| 用于重置密码的电子邮箱收取的验证码。|
|newPassword|string|重置的新密码|

**Response Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|email|string| 用于收取验证码的电子邮箱地址|



**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|发送没有验证码的请求 |`INVALID_PARAMETERS`| `{"error": "INVALID_PARAMETERS","message":"Code is missing."}`|                                                                                                                                
|用户创建的密码小于六个字符|`WEAK_PASSWORD`|`{"error": "WEAK_PASSWORD","message": "Password should be at least 6 characters"}`|   

***                 

### <a name="pacp"></a>POST /authentication/change-password


修改用户密码。如果用户更新密码成功，则所有现有的会话令牌(sessionToken)都将失效。在响应中(response)返回一个新的会话令牌(sessionToken)。

**Authorization**

需要用户access token才能调用此API。只有用户可以将外部ID提供者(ID providers)链接/取消链接到他们自己的帐户。传递access token作为标准Authorization header。

**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|password|string| 目前的密码|
|newPassword|string|新密码|



**Response Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|userId|string| 该认证用户的ID|
|email|string|用户的电子邮件地址|
|user|[User](#user)|返回登录的用户。在externalIds字段中，仅返回当前的外部ID提供者(provider)。|


**Error Responses**


| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|密码不合法时会收到以下response |`INVALID_CREDENTIALS`| `{"error": "INVALID_CREDENTIALS","message": "Email/password is invalid."}`|                                                                                                                                
|当用户尚未通过电子邮件/密码验证|`PASSWORD_AUTH_NOT_SETUP`|`{"error": "PASSWORD_AUTH_NOT_SETUP","message": "User does not have password setup."}`|  

***                  

### <a name="pav"></a>POST /authentication/verify-email

触发用户的邮件认证


**Authorization**

这是一个公开(public)API。不需要Authorization header。

**Request Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|email|string| 用于认证的电子邮件地址|
|locale|string|(可选)电子区域设置|


**Response**

没有 response body.

**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|电子邮件不合法时 |INVALID_CREDENTIALS| `{"error": "INVALID_CREDENTIALS","message": "Invalid email address."}`|   

***                                                                                                                             

#### <a name="pavc"></a>POST /authentication/verify-email/confirm

通过使用发送给用户的代码来验证电子邮件。


**Authorization**

这是一个公开(public)API。不需要Authorization header。


**Request Body**


| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|code|string| 验证电子邮件代码。|


**Response Body**

| **Field Path** | **类型** |**描述**|
| ------ | ------ | ------ | 
|email|string| 认证代码的电子邮件地址|


**Error Responses**

| **Scenario** | **Error Code** | **Example Response JSON** |
| ------ | ------ | ------ |
|发送没有验证码的请求 |`INVALID_PARAMETERS`| `{"error": "INVALID_PARAMETERS","message":"Code is missing."}`|  


                                                                                                                              

## <a name="authoapis"></a>授权 APIs

就字段详细信息而言，所有授权API均遵循OAuth2约定。有关此信息的更多信息，请查看官方的[OAuth2.0标准](https://tools.ietf.org/html/rfc6749)。

```进一步 notes: ``` 

- 当前，我们支持以下OAuth流程：
  + [Auth code flow](https://tools.ietf.org/html/rfc6749#section-1.3.1) 
     + 包括本机应用程序的[PKCE授权流程](https://tools.ietf.org/html/rfc7636)
  + [Refresh token flow](https://tools.ietf.org/html/rfc6749#section-1.5)
- 在以下情况下，我们的OAuth2授权API在以下情况可```扩展```为接收```idToken```：
  + 从身份认证API接受idToken用于跳过OAuth flow中的身份验证步骤。
  + 这样，用户可以将自定义UI与授权完全分开。

在UAS(用户身份验证服务)中，OAuth客户端有多个固定的OAuth 权限。下表显示了作用域及其相应的权限。

| **Scope** | **Permission** | **Description** |
| ------ | ------ | ------ |
|identity.user |`userauth.authn.link` `userauth.authn.unlink` `userauth.authn.create-credential` `userauth.authn.change-password` `userauth.authn.verify-email` `userauth.user.get` `userauth.user.update`|此权限允许用户更新自己的基本信息|
|identity.readonly-admin|  `userauth.user.list` `userauth.user.get`|  该权限具有对应整个ID域(ID domain)的读权限。```注意：```该权限不该赋给Apps的oauth客户端。                                                                                                                                                                            
|identity.user-admin|`userauth.authn.unlink` `userauth.authn.verify-email` `userauth.user.list` `userauth.user.get` `userauth.user.update` `userauth.user.disable` `userauth.user.enable` `userauth.user.delete`|该权限对于list enable/disable 这些apis拥有全部权限, 但是它不能用来修改ID domain setting, 注意由于reset-password是公共API，因此该权限无法调用此API。```注意：```该权限不该赋给Apps的oauth客户端。|


此外，还支持以下标准权限范围：

 + openid：存在此权限范围时，token endpoint将返回id_token。
 + offline：存在此权限范围时，token endpoint将返回refresh_token。
我们目前不处理[OIDC Basic Client Implementer's Guide](https://openid.net/specs/openid-connect-basic-1_0.html#Scopes) 草案中定义的其他标准声明（包括个人资料，电子邮件，地址，电话）。

### <a name="goa"></a>GET /oauth2/auth
OAuth 2.0 [authorization endpoint](https://tools.ietf.org/html/rfc6749#section-3.1)

### <a name="got"></a>GET /oauth2/token
OAuth 2.0 [token endpoint](https://tools.ietf.org/html/rfc6749#section-3.2)
### <a name="gwo"></a>GET /.well-known/openid-configuration

公共endpoint获取 [OpenID Connect (OIDC) configuration](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)

### <a name="gwj"></a>GET /.well-known/jwks.json

公共endpoint获取 [JSON Web Key specification](https://tools.ietf.org/html/rfc7517) 

***


## <a name="manageapi"></a>用户管理API

### <a name="gu"></a>GET /users

列出满足参数所有用户。Paging support with continuation token.具有连续令牌的分页支持。

**Authorization**

需要访问令牌(acces token)才能调用此API。传递访问令牌作为标准Authorization header。需要满足```identity.user```的范围。关于此请参考[Access Token](#accesstoken)

**Request 语句参数**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|maxResults |integer|（可选）页面大小。如果未指定，则默认为1000。值不能超过1000|
|pageToken|  string|  （可选）下一页标记token。如果未指定，则从头开始返回所有用户。|                                                                                                                                                                            
|userEmailAddress|string|（可选）用户电子邮件地址。如果未设置，则返回maxResults数量的用户的电子邮件地址以及pageToken参数。
    

**Request Body**
没有 request body.

**Response Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|users |[][User]()|当前页面返回的所有用户|
|nextPageToken|  string|  （可选）下一页标记token。如果未指定，则从头开始返回所有用户。|                                                                                                                                                                            
***
### <a name="gui"></a>GET /users/{id}

获取用户的信息

**Authorization**

需要访问令牌(acces token)才能调用此API。传递访问令牌作为标准Authorization header。需要满足```identity.user```的范围。


**Request 路径参数**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|id |string|用户的ID|


**Request Body**

没有 request body.

**Response Body**


| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| |[User](#user)|返回用户的信息|

***


### <a name="pui"></a>PUT /users/{id}

更新用户信息

**Authorization**

需要访问令牌(acces token)才能调用此API。传递访问令牌作为标准Authorization header。且需要满足```identity.user```的权限。


**请求路径参数**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| id|string|用户的ID|



**Request Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| displayName|string|(可选)用户显示的名字|
|photoUrl|string|如果用户设置了照片返回照片的URL|



**Response Body**


| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| |[User](#user)|返回用户的信息|

***

### <a name="puid"></a>POST /users/{id}/disable

禁用用户。禁用用户后，该用户将无法进行身份验证。

**Authorization**

需要访问令牌(acces token)才能调用此API。传递访问令牌作为标准Authorization header。且需要满足```identity.user-admin```的权限。

**请求路径参数**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| id|string|用户的ID|


**Request Body**

没有 request body.

**Response Body**


| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| |[User](#user)|返回用户的信息|

***

### <a name="puie"></a>POST /users/{id}/enable
启用用户

**Authorization**

需要访问令牌(acces token)才能调用此API。传递访问令牌作为标准Authorization header。且需要满足```identity.user-admin```的权限。


**请求路径参数**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| id|string|用户的ID|


**Request Body**

没有 request body.

**Response Body**


| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| |[User](#user)|返回用户的信息|

***

### <a name="dui"></a>DELETE /users/{id}
删除用户

**Authorization**

需要访问令牌(acces token)才能调用此API。传递访问令牌作为标准Authorization header。且需要满足```identity.user-admin```的权限。


**请求路径参数**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
| id|string|用户的ID|


**Request Body**

没有 request body.

**Response Body**

没有 Response Body

## <a name="datamodel"></a>核心数据模型
### <a name="user"></a>User

用户是指使用Unity身份服务登录的玩家。

| **Field Path** | **类型** |**Notes**|**描述** |
| ------ | ------ | ------ | ------ |
| id|string|产生的|用户ID。该ID在ID 域(ID Domain)中是唯一的。|
|idDomain|string| |指用户id所在的域|
|disabled|boolean| |用户是否有禁用|
|displayName|string| |用户显示的名字|
|email|string| |用户的主要电子邮件（如果已设置)。|
|emailVerified|boolean| |电子邮件是否验证过|
|metadata|object| |有关用户的其他元数据|
|photoUrl|string| |照片的URL,如果有设置|
|customClaims|object| |包含在用户ID令牌(ID tokens)中的其他声明。|
|externalIds|[]UserExternalId| |链接的外部ID provide信息的列表。列表本身不限制与此用户关联的外部帐户的数量。在当前版本中，我们仅允许每种类型一个外部帐户与用户账号链接。例如，您不能链接两个不同的Facebook ID到你的同一账户。|
|... providerId|string| |使用外部ID provider提供ID 的链接|
|... displayName|string| |链接的provider 显示的名字|
|... email|string| |链接provider 的email|
|... phoneNumber|string| |链接provider 的电话号|
|... photoUrl|string| |来自链接到的provider 的照片URL|
|... externalId|string| |来自外部provider 的用户ID|

## <a name="tokenr"></a>Token 参考

### <a name="accesstoken"></a>Access Token

访问令牌是短暂的令牌（约1小时），通常以JWT格式授予对资源的访问权限。

<font color=red>**获取用户管理API以及Admin API的```iddomain.admin_token```权限的Access Token：** </font>

请参考以下例子。

**1.首先创建某个Iddomain 下Admin OAuth Client**

*POST /oauth2/clients*

    
 **Authorization**
    
 该Token 可以在editor复制得到。在**project setting**->**Unity UserAuth**->**Copy Token**
 
 **Request Body**
 
| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|client_name|string|创建一个name(自定义)|
| id_domain|string|某个Iddomain|
|grant_types|string|值为client_credentials|
|response_types|string|值为token|
|scope|string|值为identity.admin|

例如：

`{"client_name": "test",
"id_domain": "1ed2a173-f21d-4320-86c4-b8a5f8b44c8a",
"grant_types": [
        "client_credentials"
    ],
     "response_types": [
        "token"
    ], "scope": "identity.admin"}`


**Response Body**

| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|client_id|string|返回|
| client_name|string|自定义创建的name|
|client_secret|string|返回|
|id_domain|string|当前的iddomain|
|grant_types|string|返回|
|response_types|string|返回|
|scope|string|返回|
|token_endpoint_auth_method |string|返回|

例如：

`{
    "client_id": "34f644c5-9f30-42cf-85a9-75ee63281cd1",
    "client_name": "test",
    "client_secret": "EfCDQXaLt0FmRdhBXs-jACjKx_DiTyCFUZC9X5qSu0I",
    "id_domain": "1ed2a173-f21d-4320-86c4-b8a5f8b44c8a",
    "grant_types": [
        "client_credentials"
    ],
    "response_types": [
        "token"
    ],
    "scope": "identity.admin",
    "token_endpoint_auth_method": "client_secret_post"
}`

**2.获取到某个iddomain 下的Admin Token**

*POST /oauth2/token*

**Authorization**

无

**Request Body**
 
| **Field Path** | **类型** | **描述** |
| ------ | ------ | ------ |
|grant_type|string|值为client_credentials|
| scope|string|值为identity.admin|
|client_id|string|值为上面获取的client_id|
|client_secret|string|值为上面获取的client_secret|

例如：
`{
	"grant_type": "client_credentials",
	"scope": "identity.admin",
	"client_id": "d70b7628-f33f-4d42-a07f-1a4065dc1deb",
	"client_secret": "BVubFpS5ksmADCQSZpeJ3wT6yTX2TwOD2q8GavvczAE"
}`

**Response Body**

获取到的access_token 即为当前```iddomain.admin_token```权限的access token

***




    
    



 

### <a name="idtoken"></a>ID Token

ID令牌是一种安全令牌，其中包含使用客户端时授权服务器对终端的身份验证的内容以及可能的其他请求的内容。 ID令牌表示为JSON Web
令牌（JWT）。

### <a name="refreshtoken"></a>Refresh Token

刷新令牌是寿命很长的令牌（约1个月），其可以获取到新access token。

### <a name="sessiontoken"></a>Session Token

会话令牌用于存储登录会话(login session)在客户端上。它通常不会过期。以下条件
将使会话令牌到期：

+ 用户被禁用或删除。
+ 用户有一个重大的帐户更改。例如：更改或重置密码。








