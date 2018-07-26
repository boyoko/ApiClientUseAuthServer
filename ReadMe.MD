### 建立一个web api项目

1. 新建asp.net web项目，选择 .net core 2.1 WebApi 模板
2. 为Web Api添加Swagger帮助页面
    ```
    Install-Package Swashbuckle.AspNetCore
    ```
3. 在Startup的ConfigureServices注册并配置Swagger
    ```
    // Register the Swagger generator, defining one or more Swagger documents
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new Info { Title = "My API", Version = "v1" });
    });
    ```
4. 在StartUp的Configure方法使用Swagger中间件
    ```
    // Enable middleware to serve generated Swagger as a JSON endpoint.
    app.UseSwagger();

    // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.), specifying the Swagger JSON endpoint.
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    });
    ```
5. 添加库IdentityServer4.AccessTokenValidation

    **webapi配置identity server就需要对token进行验证, 这个库就是对access token进行验证的. 通过nuget安装:**
    ```
    Install-Package IdentityServer4.AccessTokenValidation
    ```
6. 在Startup的ConfigureServices里面注册配置:
    ```
    services.AddMvcCore()
                .AddAuthorization()
                .AddJsonFormatters();

    services.AddAuthentication("Bearer")
        .AddIdentityServerAuthentication(options =>
        {
            options.RequireHttpsMetadata = false;
            options.Authority ="http://localhost:6000";  //我使用的端口是6000
            options.ApiName = "Album";
        });
    ```
7. 在Startup的Configure方法里配置Authentication中间件
    ```
    app.UseAuthentication();
    ```
    **这句话就是在把验证中间件添加到管道里, 这样每次请求就会调用验证服务了, 一定要在UserMvc()之前调用.**

8. 打开ValuesController, 在Controller上面添加[Authorize]属性,先运行AuthServer 服务, 然后运行 webapi,会自动打开这个网址: http://localhost:7000/api/values ,我修改了launchSettings.json 中的端口，你的端口可能跟我不一样
9. 运行结果显示401, 显示该请求为UnAuthorized,所以我们首先需要获取到一个token,使用pastman,用post方式请求 http://localhost:6000/connect/token , body中的请求参数格式选择：x-www-form-urlencoded ,参数及参数值如下表：
 
      参数         |      参数值 
  ----------------|-------------------- 
    client_id     |    Album   
    client_secret |     secret    
    grant_type    |    client_credentials

10. 复制一下 access_token的值. 回到api/values的那个请求, 在Header里面增加一个key,输入Authorization , value 列输入Bearer, 还有一个空格,把access_token贴到后面, 如下表:
    
    Key           |      value    
----------------- | -----------------
    Authorization | Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6ImZjMWFlZTkyZDEwYWZhOGU1NTFmMDc5YzExNzUzODkxIiwidHlwIjoiSldUIn0.eyJuYmYiOjE1MzI1OTI0ODQsImV4cCI6MTUzMjU5NjA4NCwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo2MDAwIiwiYXVkIjpbImh0dHA6Ly9sb2NhbGhvc3Q6NjAwMC9yZXNvdXJjZXMiLCJBbGJ1bSJdLCJjbGllbnRfaWQiOiJBbGJ1bSIsInNjb3BlIjpbIkFsYnVtIl19.A471EFMO24tNeEnXOfZQ4PDljSR7iyQehB0eUEqRtiE99YN4sF6MMbACWO3ab0l_8ggT78ewsBHfZJByiN10CyWCVJk7n5h4lPTcf5bBfvI1W1SkJ4sTenGIUIeWdogQ4hdxRa5ghn5PsXd-h4DaWB3mArLqncON3qR_Pl7M4XwdWTLBSKk1jp6KpzlScFm5cXoM5dNdCImEwEECw9tResKGCTrS-ISTxw2bkyLLx4C17Cgedd22nXpoan3u4XNAYTI2eOOL9jxGJ1obgYaZJK8Htve3ofNcCmMCPxQNGcAUc2gnB-6f2I9aGfFeWVtNHyDzgyJm0zEZxTA6VqRBpg