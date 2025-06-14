# 配置管理中心集群 & 动态配置刷新 测试指南

## 1. 架构概述

### 配置管理中心集群
- **config-server**: 端口 8888 (主配置服务器)
- **config-server2**: 端口 8889 (配置服务器集群节点)
- **消息总线**: RabbitMQ (配置刷新事件分发)
- **服务注册**: Eureka (服务发现和负载均衡)

### 客户端服务
- **consumer-service**: 端口 9001 (配置消费者)
- **支持功能**: 动态配置刷新、消息总线、配置监控

## 2. 启动顺序

### 2.1 基础服务启动
```bash
# 1. 启动 RabbitMQ (确保端口 5672 可用)
# 2. 启动 Eureka Server (端口 8761, 8762)
# 3. 启动配置服务器集群
java -jar config-server.jar         # 端口 8888
java -jar config-server2.jar        # 端口 8889
# 4. 启动消费者服务
java -jar consumer-service.jar      # 端口 9001
```

### 2.2 验证服务启动
```bash
# 检查 Eureka 注册中心
curl http://localhost:8761/eureka/apps

# 检查配置服务器状态
curl http://localhost:8888/actuator/health
curl http://localhost:8889/actuator/health

# 检查消费者服务状态
curl http://localhost:9001/actuator/health
```

## 3. 配置管理功能测试

### 3.1 获取当前配置
```bash
# 获取完整配置信息
curl http://localhost:9001/config-test/current

# 获取配置测试结果
curl http://localhost:9001/config-test/test

# 获取配置刷新状态
curl http://localhost:9001/config-test/refresh-status

# 健康检查
curl http://localhost:9001/config-test/health
```

### 3.2 配置服务器集群测试
```bash
# 从不同配置服务器获取配置
curl http://localhost:8888/consumer-service/dev
curl http://localhost:8889/consumer-service/dev

# 检查配置服务器状态
curl http://localhost:8888/config-refresh/status
curl http://localhost:8889/config-refresh/status
```

## 4. 动态配置刷新测试

### 4.1 修改配置文件
1. 修改 `consumer-service.yml` 中的配置项：
   ```yaml
   consumer:
     config:
       version: 2.2.0
       customMessage: "🚀 动态配置刷新测试 - 版本2.2.0"
       debugMode: false
   ```

2. 提交配置变更到 Git 仓库

### 4.2 触发配置刷新

#### 方法1: 手动刷新本地配置
```bash
# 刷新消费者服务本地配置
curl -X POST http://localhost:9001/actuator/refresh

# 或使用自定义端点
curl -X POST http://localhost:9001/config-test/refresh-local
```

#### 方法2: 通过消息总线全局刷新
```bash
# 从配置服务器触发全局刷新
curl -X POST http://localhost:8888/actuator/bus-refresh

# 或使用自定义端点
curl -X POST http://localhost:8888/config-refresh/global

# 从消费者服务触发全局刷新
curl -X POST http://localhost:9001/config-test/refresh-global
```

#### 方法3: 刷新指定服务
```bash
# 只刷新 consumer-service
curl -X POST http://localhost:8888/config-refresh/service/consumer-service

# 或通过总线端点
curl -X POST "http://localhost:8888/actuator/bus-refresh?destination=consumer-service:**"
```

### 4.3 验证配置刷新结果
```bash
# 检查配置是否更新
curl http://localhost:9001/config-test/current | grep version
curl http://localhost:9001/config-test/current | grep customMessage

# 检查配置测试结果
curl http://localhost:9001/config-test/test
```

## 5. 配置刷新事件监控

### 5.1 查看日志
监控以下日志信息：
- 配置刷新事件接收
- 配置项变更记录
- 消息总线事件传播
- 配置加载状态

### 5.2 监控端点
```bash
# 查看配置属性
curl http://localhost:9001/actuator/configprops

# 查看环境信息
curl http://localhost:9001/actuator/env

# 查看应用信息
curl http://localhost:9001/actuator/info

# 查看健康状态
curl http://localhost:9001/actuator/health
```

## 6. 高级测试场景

### 6.1 配置服务器故障转移测试
1. 停止 config-server (8888)
2. 验证 consumer-service 是否自动切换到 config-server2 (8889)
3. 触发配置刷新，验证功能正常

### 6.2 消息总线故障测试
1. 停止 RabbitMQ
2. 验证本地配置刷新仍然可用
3. 重启 RabbitMQ 后验证消息总线恢复

### 6.3 批量配置更新测试
1. 同时修改多个配置项
2. 触发一次全局刷新
3. 验证所有配置项都正确更新

## 7. 性能监控

### 7.1 配置刷新性能
- 监控配置刷新时间
- 观察内存使用变化
- 检查服务响应时间

### 7.2 集群负载均衡
- 验证配置请求在集群节点间的分布
- 监控各节点的负载情况

## 8. 故障排查

### 8.1 常见问题
1. **配置刷新失败**
   - 检查 RabbitMQ 连接
   - 验证配置服务器可访问性
   - 查看应用日志

2. **配置不生效**
   - 确认 @RefreshScope 注解
   - 检查配置文件语法
   - 验证配置项名称

3. **消息总线异常**
   - 检查 RabbitMQ 状态
   - 验证消息总线配置
   - 查看总线事件日志

### 8.2 日志分析
关键日志模式：
- `接收到远程配置刷新事件`
- `配置刷新成功，刷新的配置项`
- `全局配置刷新信号已发送`
- `配置项变更`

## 9. 测试清单

### 9.1 基本功能测试
- [ ] 配置服务器集群启动正常
- [ ] 消费者服务可获取配置
- [ ] 配置内容正确显示
- [ ] 健康检查通过

### 9.2 动态刷新测试
- [ ] 本地配置刷新成功
- [ ] 全局配置刷新成功
- [ ] 指定服务刷新成功
- [ ] 配置变更正确反映

### 9.3 高可用测试
- [ ] 配置服务器故障转移
- [ ] 消息总线故障恢复
- [ ] 服务重启配置恢复

### 9.4 监控测试
- [ ] 配置刷新事件记录
- [ ] 性能指标正常
- [ ] 错误告警机制

## 10. 扩展功能

### 10.1 配置加密
- 敏感配置项加密存储
- 动态解密机制

### 10.2 配置版本管理
- 配置变更历史追踪
- 配置回滚机制

### 10.3 多环境配置
- 开发、测试、生产环境隔离
- 环境特定配置覆盖 