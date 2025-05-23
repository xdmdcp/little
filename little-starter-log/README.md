# little-starter-log

little-starter-log 是一个基于 Spring Boot 的轻量级日志记录模块，旨在为 Web 应用提供统一的 API 日志记录能力。通过 AOP
实现对控制器方法的自动监控，并支持灵活配置和扩展发布逻辑。

## 📌 功能概述

- API 日志自动记录：使用 @ApiLog 注解标记需要记录的方法。
- 请求上下文采集：包括 IP、UA、HTTP 方法、路径、参数等。
- 耗时统计与异常捕获：记录接口执行时间及异常信息。
- 异步日志发布：默认实现打印 JSON 格式日志，可扩展至数据库、消息队列等。
- 配置化控制：
    - 启用/禁用日志功能
    - 排除特定路径（如 /health）
- Spring Boot 自动装配支持：开箱即用，零配置即可启用日志记录。

## 🧱 模块结构

```
little-starter-log/
├── src/
│ ├── main/
│ │ ├── java/cn/chenlijian/little/starter/log/
│ │ │ ├── aspect/ # AOP 切面处理类
│ │ │ │ └── ApiLogAspect.java # 核心切面逻辑 
│ │ │ ├── props/ # 配置属性类 
│ │ │ │ ├── LittleLogProperties.java # 支持 little.log.* 开头的配置项 
│ │ │ │ └── SamplingProperties.java # 采样率和错误日志配置 
│ │ │ ├── publisher/ # 日志发布器接口及默认实现 
│ │ │ │ ├── LogPublisher.java 
│ │ │ │ └── DefaultLogPublisher.java 
│ │ │ ├── sampling/ # 采样策略接口和实现 
│ │ │ │ ├── strategy/ # 具体采样策略 
│ │ │ │ │ ├── ErrorForcedSamplingStrategy.java # 强制记录异常请求 
│ │ │ │ │ └── RandomSamplingStrategy.java # 随机采样策略 
│ │ │ │ └── SamplingStrategy.java # 采样策略接口 
│ │ │ ├── utils/ # 工具类 
│ │ │ │ ├── LogUtil.java # 日志构建工具 
│ │ │ │ └── WebUtil.java # Web 请求相关工具方法 
│ │ │ └── LittleLogAutoConfiguration.java # Spring Boot 自动装配类 
│ │ └── resources/META-INF/ 
│ │ └── spring.factories # Spring Boot SPI 配置文件 
│ └── test/java # 测试代码（当前为空） 
└── pom.xml # Maven 构建配置
```

## 🛠️ 使用说明

### 1. 引入依赖

在 pom.xml 文件中添加如下依赖：

```xml
<dependency>
  <groupId>cn.chenlijian.little</groupId>
  <artifactId>little-starter-log</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>

```

### 2. 配置选项（application.yml）

```yaml
little:
  log:
    enabled: true
    exclude-paths:
      - "/actuator/**"
      - "/login"
    # 日志采样功能 
    sampling:
      rate: 1.0 # 设置采样率为 100%
      include-error: true # 强制记录异常请求
```

### 3. 添加注解

在需要记录日志的 Controller 方法上添加 @ApiLog 注解：

```java

@GetMapping("/hello")
@ApiLog("用户访问首页")
public String sayHello() {
    return "Hello World!";
}
```

## ⚙️ 高级定制

### 自定义日志存储

实现 LogPublisher 接口并替换默认实现：

```java

@Component
public class DatabaseLogPublisher implements LogPublisher {
    @Override
    public void publish(ApiLogDTO record) {
        // 存储到数据库或其他处理逻辑
    }
}
```

## 📦 依赖列表

| 依赖项                     | 版本     | 说明           |
|-------------------------|--------|--------------|
| spring-boot-starter-aop | 3.2.x  | 提供 AOP 支持    |
| spring-web              | 6.1.x  | Web 相关组件     |
| jakarta.servlet-api     | 6.0.x  | Servlet 规范支持 |
| hutool-all              | 5.8.x  | 工具类库         |
| lombok                  | 1.18.x | 简化 POJO 编写   |

## ⚙️ 新增功能说明

### 采样日志记录

本模块新增了日志采样功能，允许根据配置的采样率随机记录日志，并且可以强制记录发生异常的请求。

- **采样率配置**：通过 `sampling.rate` 参数配置，表示日志记录的概率。例如：0.5 表示 50% 的概率会记录日志，默认值为 1（即 100% 记录）。
- **强制记录异常请求**：通过 `sampling.include-error` 参数配置，如果设置为 `true`，则在发生异常时总是记录日志，无论采样率如何。
- **采样策略**：本模块提供了两种采样策略：随机采样和强制记录异常请求。默认为随机采样。可以通过实现：[SamplingStrategy.java](src/main/java/cn/chenlijian/little/starter/log/sampling/SamplingStrategy.java) 实现自定义采样策略。
