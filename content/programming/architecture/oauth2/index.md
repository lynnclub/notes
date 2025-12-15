---
title: "Oauth2"
type: "docs"
weight: 1
---

## 授权模式

OAuth 2.0 定义了几种授权模式，用于在客户端和资源所有者之间进行身份验证和授权。

1. **授权码模式（Authorization Code Grant）：**

   - **使用场景：** 适用于有服务器的应用，如 Web 应用。
   - **流程：** 客户端将用户导向认证服务器，用户登录并授权后，认证服务器返回一个授权码，然后客户端使用该授权码向令牌端点请求访问令牌。
   - **使用方式：** 客户端需要通过 HTTP 重定向将用户引导到认证服务器，并在返回的回调 URL 中获取授权码。
   - **请求参数示例：**
     ```http
     # 步骤1：获取授权码
     GET /oauth/authorize?response_type=code
         &client_id=YOUR_CLIENT_ID
         &redirect_uri=https://your-app.com/callback
         &scope=read write
         &state=xyz123
     
     # 步骤2：用授权码换取访问令牌
     POST /oauth/token
     Content-Type: application/x-www-form-urlencoded
     
     grant_type=authorization_code
     &code=AUTHORIZATION_CODE
     &redirect_uri=https://your-app.com/callback
     &client_id=YOUR_CLIENT_ID
     &client_secret=YOUR_CLIENT_SECRET
     ```

2. **隐式授权模式（Implicit Grant）：**

   - **使用场景：** 适用于纯前端应用，如单页面应用（SPA）。
   - **流程：** 客户端在重定向 URI 的片段中接收访问令牌，而不是通过授权码交换的方式。
   - **使用方式：** 客户端通过重定向 URI 接收访问令牌，无需进行授权码交换。
   - **请求参数示例：**
     ```http
     # 直接获取访问令牌
     GET /oauth/authorize?response_type=token
         &client_id=YOUR_CLIENT_ID
         &redirect_uri=https://your-app.com/callback
         &scope=read
         &state=xyz123
     
     # 服务器重定向返回（通过 URL 片段）
     https://your-app.com/callback#access_token=ACCESS_TOKEN
         &token_type=Bearer
         &expires_in=3600
         &state=xyz123
     ```

3. **密码模式（Resource Owner Password Credentials Grant）：**

   - **使用场景：** 适用于受信任的客户端，如移动应用或命令行工具。
   - **流程：** 客户端直接使用用户的用户名和密码向认证服务器请求访问令牌。
   - **使用方式：** 客户端通过直接向认证服务器发送用户凭证（用户名和密码）来获取访问令牌。
   - **请求参数示例：**
     ```http
     POST /oauth/token
     Content-Type: application/x-www-form-urlencoded
     
     grant_type=password
     &username=USER_USERNAME
     &password=USER_PASSWORD
     &client_id=YOUR_CLIENT_ID
     &client_secret=YOUR_CLIENT_SECRET
     &scope=read write
     ```

4. **客户端凭证模式（Client Credentials Grant）：**

   - **使用场景：** 适用于客户端自身而不是代表用户请求访问令牌的情况，如后台服务。
   - **流程：** 客户端使用自己的凭证（客户端 ID 和客户端密钥）向认证服务器请求访问令牌。
   - **使用方式：** 客户端通过使用自身的凭证向认证服务器请求访问令牌，无需用户参与。
   - **请求参数示例：**
     ```http
     POST /oauth/token
     Content-Type: application/x-www-form-urlencoded
     
     grant_type=client_credentials
     &client_id=YOUR_CLIENT_ID
     &client_secret=YOUR_CLIENT_SECRET
     &scope=read
     ```

5. **刷新令牌模式（Refresh Token Grant）：**
   - **使用场景：** 用于通过刷新令牌获取新的访问令牌，而无需用户重新授权。
   - **流程：** 客户端使用刷新令牌向认证服务器请求新的访问令牌。
   - **使用方式：** 客户端在访问令牌过期时，通过刷新令牌向认证服务器请求新的访问令牌。
   - **请求参数示例：**
     ```http
     POST /oauth/token
     Content-Type: application/x-www-form-urlencoded
     
     grant_type=refresh_token
     &refresh_token=REFRESH_TOKEN
     &client_id=YOUR_CLIENT_ID
     &client_secret=YOUR_CLIENT_SECRET
     &scope=read write
     ```

在实际应用中，选择合适的授权模式取决于你的应用类型和安全需求。
