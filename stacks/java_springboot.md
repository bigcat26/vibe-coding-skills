---
name: stack-java-springboot
description: Java 后端开发技术栈标准。基于 Spring Boot / Spring Cloud 微服务体系，使用 Gradle (Kotlin DSL) 构建，集成 Checkstyle (代码规范)、JaCoCo (覆盖率) 和 JUnit 5 (测试)。
version: 1.0.0
tags: [java, springboot, springcloud, gradle, junit5, mybatis-plus]
---

# Java / Spring Boot / Spring Cloud 开发技术栈规范

本文档定义了 Vibe Coding 体系下 Java 后端项目的强制性技术栈和开发标准。核心理念是微服务架构、契约优先设计和严格的分层约束。

## 工具链标准

Agent 必须使用以下工具，除非用户明确指定其他替代方案：

- **JDK 版本**: `Java 17` (LTS)，新项目推荐 `Java 21` (LTS)
- **构建工具**: `Gradle` (Kotlin DSL `build.gradle.kts`)，替代 Maven
- **框架**: `Spring Boot 3.x` + `Spring Cloud 2023.x`
- **API 文档**: `SpringDoc OpenAPI` (Swagger UI)
- **ORM / 数据访问**: `MyBatis-Plus` (首选) 或 `Spring Data JPA`
- **数据库迁移**: `Flyway`
- **代码检查**: `Checkstyle` + `SpotBugs`
- **代码格式化**: `Spotless` (Google Java Format)
- **测试框架**: `JUnit 5` + `Mockito` + `Testcontainers`
- **覆盖率**: `JaCoCo`
- **日志**: `SLF4J` + `Logback` (Spring Boot 默认)

## 项目结构 (Layout)

### 单体 Spring Boot 项目

- `project_name/`
  - `src/`
    - `main/`
      - `java/com/example/project_name/`
        - `ProjectNameApplication.java`
        - `config/` (配置类)
        - `controller/` (REST 控制器)
        - `service/` (业务逻辑接口)
        - `service/impl/` (业务逻辑实现)
        - `mapper/` (MyBatis-Plus Mapper 接口)
        - `model/`
          - `entity/` (数据库实体)
          - `dto/` (数据传输对象)
          - `vo/` (视图对象)
        - `exception/` (自定义异常)
        - `common/` (通用工具类、常量、枚举)
      - `resources/`
        - `application.yml`
        - `application-dev.yml`
        - `application-prod.yml`
        - `db/migration/` (Flyway 迁移脚本)
        - `mapper/` (MyBatis XML 映射文件，如使用)
    - `test/`
      - `java/com/example/project_name/`
        - `controller/` (Controller 层测试)
        - `service/` (Service 层测试)
        - `mapper/` (Mapper 层测试)
  - `build.gradle.kts`
  - `settings.gradle.kts`
  - `gradle.properties`
  - `Dockerfile`
  - `.gitignore`
  - `README.md`

### Spring Cloud 微服务项目

- `project_name/`
  - `project-gateway/` (API 网关，Spring Cloud Gateway)
  - `project-auth/` (认证服务)
  - `project-common/` (公共模块：工具类、通用 DTO)
  - `project-service-a/` (业务服务 A)
  - `project-service-b/` (业务服务 B)
  - `build.gradle.kts` (根构建文件)
  - `settings.gradle.kts` (包含所有子模块)
  - `gradle.properties`
  - `docker-compose.yml` (本地开发环境)
  - `.gitignore`
  - `README.md`

## 初始化工作流

当初始化一个新的 Java 项目时，按以下顺序执行：

- 使用 Spring Initializr 或手动创建 Gradle 项目结构。
- 配置 `build.gradle.kts`：
  - 添加 Spring Boot 插件和依赖管理。
  - 添加核心依赖：`spring-boot-starter-web`, `spring-boot-starter-validation`。
  - 添加开发依赖：`spring-boot-devtools`, `lombok`。
  - 添加测试依赖：`spring-boot-starter-test`, `testcontainers`。
  - 配置 Spotless 插件 (Google Java Format)。
  - 配置 JaCoCo 插件 (覆盖率 >= 80%)。
- 配置 `application.yml`：
  - 设置 server port、application name。
  - 配置 profile 激活策略。
- 创建 `.gitignore`：
  - 必须包含 `.gradle/`, `build/`, `.idea/`, `*.iml`, `*.class`, `*.jar`, `*.log`, `out/`。

## 配置标准

### build.gradle.kts 核心配置

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.3.0"
    id("io.spring.dependency-management") version "1.1.5"
    id("com.diffplug.spotless") version "6.25.0"
    jacoco
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

spotless {
    java {
        googleJavaFormat()
        removeUnusedImports()
    }
}

jacoco {
    toolVersion = "0.8.12"
}

tasks.jacocoTestReport {
    dependsOn(tasks.test)
    reports {
        xml.required = true
        html.required = true
    }
}

tasks.jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = "0.80".toBigDecimal()
            }
        }
    }
}
```

### Spring Cloud 微服务配置

当项目为微服务架构时，额外引入以下组件：

- **服务注册与发现**: `Spring Cloud Alibaba Nacos` 或 `Consul`
- **配置中心**: `Nacos Config` 或 `Spring Cloud Config`
- **服务网关**: `Spring Cloud Gateway`
- **服务间调用**: `OpenFeign` + `Spring Cloud LoadBalancer`
- **熔断降级**: `Sentinel` 或 `Resilience4j`
- **分布式事务**: `Seata` (按需引入)

### application.yml 规范

```yaml
spring:
  application:
    name: ${project_name}
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://localhost:3306/${db_name}?useSSL=false&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

mybatis-plus:
  mapper-locations: classpath:mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl

server:
  port: 8080
```

## 开发指令集

Agent 必须使用以下命令执行开发任务：

- **构建项目**：
  - `./gradlew build`
- **运行应用**：
  - `./gradlew bootRun`
- **运行测试**：
  - `./gradlew test`
- **格式化代码**：
  - `./gradlew spotlessApply`
- **检查代码格式**：
  - `./gradlew spotlessCheck`
- **生成覆盖率报告**：
  - `./gradlew jacocoTestReport`
- **清理构建**：
  - `./gradlew clean`
- **构建 Docker 镜像**：
  - `./gradlew bootBuildImage` 或 `docker build -t project_name .`

## 编码规范

- **命名规范**：
  - 类名：`PascalCase` (如 `UserService`)
  - 方法/变量：`camelCase` (如 `getUserById`)
  - 常量：`UPPER_SNAKE_CASE` (如 `MAX_RETRY_COUNT`)
  - 包名：全小写 (如 `com.example.project.service`)
  - 数据库表名/字段名：`snake_case` (如 `user_info`, `create_time`)
- **Lombok 使用**：
  - Entity 类使用 `@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`。
  - 日志使用 `@Slf4j`，严禁手动创建 Logger。
- **DTO/VO 分离**：
  - Controller 层接收 DTO，返回 VO，严禁直接暴露 Entity。
  - 使用 `MapStruct` 或手动转换进行对象映射。
- **接口设计**：
  - RESTful API 遵循标准 HTTP 方法语义 (GET/POST/PUT/DELETE)。
  - 统一响应格式：`{ "code": 200, "message": "success", "data": {} }`。
  - 使用 `@Validated` 进行参数校验，配合 `javax.validation` 注解。
- **依赖注入**：
  - 优先使用构造器注入，严禁使用字段注入 (`@Autowired` 在字段上)。
  - Service 层面向接口编程，Controller 注入接口而非实现类。

## 错误处理

- 使用 `@RestControllerAdvice` + `@ExceptionHandler` 实现全局异常处理。
- 定义业务异常基类 `BusinessException`，包含错误码和消息。
- 严禁在 Controller 层使用 `try-catch`，统一由全局异常处理器捕获。
- 使用 SLF4J 记录异常日志，严禁使用 `e.printStackTrace()`。
- 区分业务异常（返回友好提示）和系统异常（返回通用错误信息，记录详细日志）。

## 测试规范

- **单元测试**：使用 `@ExtendWith(MockitoExtension.class)` + `@Mock` / `@InjectMocks`。
- **集成测试**：使用 `@SpringBootTest` + `@AutoConfigureMockMvc`。
- **数据库测试**：使用 `Testcontainers` 启动真实数据库实例，避免 H2 兼容性问题。
- **API 测试**：使用 `MockMvc` 或 `WebTestClient` 测试 Controller 层。
- 测试方法命名：`should_ExpectedBehavior_When_Condition` (如 `should_ReturnUser_When_IdExists`)。
