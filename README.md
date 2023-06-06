## 1、什么是 YaLingAuth？

YaLingAuth，如你所见，它仅仅是一个**第三方授权登录**的**工具类库**，让我们可以避免接触繁琐的操作流程，只需要获取对应平台的基本信息实现授权登录。
YaLingAuth 集成了诸如：Github、Gitee、支付宝、新浪微博、微信、Google、Facebook等国内外数十家第三方平台。更多请参考已集成的平台

## 2、有哪些特点？

1. **全**：已集成十多家第三方平台（国内外常用的基本都已包含），仍然还在持续扩展中（开发计划）！
2. **简**：API就是奔着最简单去设计的（见后面快速开始），尽量让您用起来没有障碍感！

## 3、有哪些功能？

- 集成国内外数十家第三方平台，实现快速接入。
- 自定义 State 缓存，支持各种分布式缓存组件。
- 自定义 OAuth 平台，更容易适配自有的 OAuth 服务。
- 自定义 Http 实现，选择权完全交给开发者，不会单独依赖某一具体实现。
- 自定义 Scope，支持更完善的授权体系。
- 更多

## 4、快速开始

### 4.1、引入依赖

```xml
<dependency>
    <groupId>io.github.codergjw</groupId>
    <artifactId>yaling-auth</artifactId>
    <version>1.0.0</version>
</dependency>
```

**latest-version** 可选：

- 稳定版：`1.0.0`
- 快照版：`1.0.0-SNAPSHOT`

注意：快照版本是功能的尝鲜，并不保证稳定性。请勿在生产环境中使用。
如何引入快照版本
如下**任选一种** HTTP 工具 依赖，_项目内如果已有，请忽略。另外需要特别注意，如果项目中已经引入了低版本的依赖，请先排除低版本依赖后，再引入高版本或者最新版本的依赖_

- hutool-http

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-http</artifactId>
    <version>5.7.7</version>
</dependency>
```

- httpclient

```xml
<dependency>
	<groupId>org.apache.httpcomponents</groupId>
  	<artifactId>httpclient</artifactId>
  	<version>4.5.13</version>
</dependency>
```

- okhttp

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>4.9.1</version>
</dependency>
```

### 4.2、调用api

#### 4.2.1、普通方式

```java
// 创建授权request
AuthRequest authRequest = new AuthGiteeRequest(AuthConfig.builder()
        .clientId("clientId")
        .clientSecret("clientSecret")
        .redirectUri("redirectUri")
        .build());
// 生成授权页面
authRequest.authorize("state");
// 授权登录后会返回code（auth_code（仅限支付宝））、state，1.8.0版本后，可以用AuthCallback类作为回调接口的参数
// 注：JustAuth默认保存state的时效为3分钟，3分钟内未使用则会自动清除过期的state
authRequest.login(callback);
```

#### 4.2.2、Builder 方式一

静态配置 AuthConfig

```java
AuthRequest authRequest = AuthRequestBuilder.builder()
    .source("github")
    .authConfig(AuthConfig.builder()
        .clientId("clientId")
        .clientSecret("clientSecret")
        .redirectUri("redirectUri")
        .build())
    .build();
// 生成授权页面
  authRequest.authorize("state");
// 授权登录后会返回code（auth_code（仅限支付宝））、state，1.8.0版本后，可以用AuthCallback类作为回调接口的参数
// 注：JustAuth默认保存state的时效为3分钟，3分钟内未使用则会自动清除过期的state
  authRequest.login(callback);
```

#### 4.2.3、Builder 方式二

动态获取并配置 AuthConfig

```java
AuthRequest authRequest = AuthRequestBuilder.builder()
    .source("gitee")
    .authConfig((source) -> {
        // 通过 source 动态获取 AuthConfig
        // 此处可以灵活的从 sql 中取配置也可以从配置文件中取配置
        return AuthConfig.builder()
            .clientId("clientId")
            .clientSecret("clientSecret")
            .redirectUri("redirectUri") // 一定要和你自己对应的平台的回调地址相关匹配
            .build();
    })
    .build();
Assert.assertTrue(authRequest instanceof AuthGiteeRequest);
// 返回的就是授权地址
System.out.println(authRequest.authorize(AuthStateUtils.createState()));
```

#### 4.2.4、Builder 方式支持自定义的平台

```java
AuthRequest authRequest = AuthRequestBuilder.builder()
    // 关键点：将自定义实现的 AuthSource 配置上
    .extendSource(AuthExtendSource.values())
    // source 对应 AuthExtendSource 中的枚举 name
    .source("other")
    // ... 其他内容不变，参考上面的示例
    .build();
```