# ELM微服务系统配置中心

这个仓库包含了ELM微服务系统的所有配置文件，通过Spring Cloud Config Server进行集中管理。

## 📁 配置文件结构

### 🌍 全局配置
- `application.yml` - 所有微服务的通用配置
- `application-dev.yml` - 开发环境专用配置
- `application-prod.yml` - 生产环境专用配置

### 🔧 微服务专用配置
- `user-service.yml` - 用户服务配置（端口: 8002）
- `business-service.yml` - 商家服务配置（端口: 8001）
- `order-service.yml` - 订单服务配置（端口: 8003）
- `food-service.yml` - 食品服务配置（端口: 8004）
- `cart-service.yml` - 购物车服务配置（端口: 8005）
- `address-service.yml` - 地址服务配置（端口: 8006）
- `consumer-service.yml` - 消费者服务配置（端口: 9001）
- `gateway-service.yml` - 网关服务配置（端口: 9000）

## 🔑 配置优先级

Spring Cloud Config的配置优先级（从高到低）：
1. `{service-name}-{profile}.yml`
2. `{service-name}.yml`
3. `application-{profile}.yml`
4. `application.yml`

## 💾 数据库配置

### 开发环境
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/elm?characterEncoding=utf-8
    username: root
    password: 2254600749@qq.com
```

### 生产环境
```yaml
spring:
  datasource:
    url: jdbc:mysql://prod-db-server:3306/elm?characterEncoding=utf-8&useSSL=true
    username: ${DB_USERNAME:elm_user}
    password: ${DB_PASSWORD:your_secure_password}
```

## 🔄 服务注册中心

### Eureka集群配置
```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
```

## 🐰 消息队列配置

### RabbitMQ配置
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

## 🛡️ 熔断器配置

使用Resilience4j进行熔断保护：
```yaml
resilience4j:
  circuitbreaker:
    instances:
      default:
        failure-rate-threshold: 50
        minimum-number-of-calls: 5
        wait-duration-in-open-state: 10s
```

## 📊 监控配置

### Actuator端点配置
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
```

## 🌐 网关路由配置

API网关路由规则：
- `/api/user/**` → user-service
- `/api/business/**` → business-service
- `/api/order/**` → order-service
- `/api/food/**` → food-service
- `/api/cart/**` → cart-service
- `/api/address/**` → address-service

## 🚀 使用方法

### 1. 启动配置中心
```bash
# 启动主配置中心
cd elm_springboot/config-server
mvn spring-boot:run

# 启动备用配置中心
cd elm_springboot/config-server2
mvn spring-boot:run
```

### 2. 微服务获取配置
微服务通过以下配置获取配置中心的配置：
```yaml
spring:
  config:
    import: optional:configserver:http://localhost:8888
```

### 3. 动态刷新配置
```bash
# 刷新所有服务配置
curl -X POST http://localhost:8888/actuator/bus-refresh

# 刷新指定服务配置
curl -X POST http://localhost:8888/actuator/bus-refresh/{service-name}
```

### 4. 验证配置
```bash
# 获取默认配置
curl http://localhost:8888/application/default

# 获取特定服务配置
curl http://localhost:8888/{service-name}/default

# 获取特定环境配置
curl http://localhost:8888/{service-name}/dev
curl http://localhost:8888/{service-name}/prod
```

## 🏷️ 环境标识

- `default` - 默认环境
- `dev` - 开发环境
- `prod` - 生产环境

## 📝 配置修改流程

1. 修改对应的配置文件
2. 提交到Git仓库
3. 通过POST请求触发配置刷新
4. 微服务自动获取最新配置

## ⚠️ 注意事项

1. **敏感信息**：生产环境的敏感信息（如数据库密码）应使用环境变量
2. **配置验证**：修改配置后要验证语法正确性
3. **回滚机制**：重要配置修改前要做好备份
4. **权限控制**：配置仓库应设置合适的访问权限

## 🔍 故障排查

### 配置获取失败
1. 检查配置中心是否正常运行
2. 验证Git仓库是否可访问
3. 确认微服务配置的配置中心地址正确

### 配置不生效
1. 检查配置文件命名是否正确
2. 验证配置语法是否正确
3. 确认是否触发了配置刷新

### 网络连接问题
1. 检查Eureka注册中心状态
2. 验证RabbitMQ连接状态
3. 确认防火墙设置 