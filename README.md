# Project Collaboration Platform

一个基于 Spring Boot 3 的项目协同管理后端，用于管理项目、任务、文件记录和操作日志。系统内置演示数据和轻量认证机制，适合快速启动、接口调试和二次扩展。

## 功能概览

- 认证鉴权：演示账号登录、JWT 签发、接口鉴权拦截
- 用户角色：内置 `ADMIN`、`MANAGER`、`MEMBER` 三类角色
- 项目管理：项目列表、项目创建、项目状态流转
- 任务协同：任务创建、状态流转、负责人筛选、优先级、截止时间
- 文件记录：记录项目文件元数据、上传人、文件大小和类型
- 操作审计：记录项目、任务、文件等关键操作，支持条件查询
- 仪表盘：聚合项目数量、任务数量、任务状态分布和操作日志数量
- 接口文档：集成 Swagger/OpenAPI
- 容器化：提供 Dockerfile、Docker Compose 和 Nginx 反向代理配置

## 技术栈

- Java 17
- Spring Boot 3
- Spring MVC
- Jakarta Validation
- springdoc-openapi
- JWT / HMAC-SHA256
- Docker / Docker Compose
- MySQL、Redis、Nginx 容器编排示例

## 架构说明

项目按业务边界拆分模块：

```text
src/main/java/com/liujianan/collab
├── auth        # 登录认证
├── audit       # 操作日志
├── common      # 统一响应、异常处理
├── config      # Web 配置与鉴权拦截器
├── dashboard   # 仪表盘汇总
├── file        # 文件记录
├── project     # 项目管理
├── security    # JWT、当前用户上下文
├── task        # 任务协同
└── user        # 用户与角色
```

当前版本使用内存 Repository 保存演示数据，便于快速启动和接口验证。Repository、Service、Controller 分层边界保持清晰，可替换为 MySQL、MyBatis-Plus、Redis 等持久化实现。

## 快速启动

```bash
mvn spring-boot:run
```

启动后访问：

- Swagger UI: http://localhost:8080/swagger-ui/index.html
- OpenAPI JSON: http://localhost:8080/v3/api-docs

## 演示账号

| 角色 | 用户名 | 密码 |
| --- | --- | --- |
| 管理员 | `admin` | `admin123` |
| 项目负责人 | `manager` | `manager123` |
| 成员 | `member` | `member123` |

## API 示例

### 登录

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"admin\",\"password\":\"admin123\"}"
```

将返回的 `token` 放入后续请求头：

```text
Authorization: Bearer <token>
```

### 仪表盘汇总

```bash
curl http://localhost:8080/api/dashboard/summary \
  -H "Authorization: Bearer <token>"
```

### 创建项目

```bash
curl -X POST http://localhost:8080/api/projects \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "{\"name\":\"协同管理系统\",\"description\":\"管理项目、任务和审计日志\"}"
```

### 创建任务

```bash
curl -X POST http://localhost:8080/api/tasks \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "{\"projectId\":1,\"title\":\"完善接口文档\",\"description\":\"补充 Swagger 和 README 示例\",\"assigneeId\":2,\"priority\":\"HIGH\",\"dueDate\":\"2026-05-10\"}"
```

### 条件查询任务

```bash
curl "http://localhost:8080/api/tasks?projectId=1&status=REVIEW" \
  -H "Authorization: Bearer <token>"
```

### 记录文件

```bash
curl -X POST http://localhost:8080/api/files \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d "{\"projectId\":1,\"filename\":\"api-design.md\",\"contentType\":\"text/markdown\",\"size\":2048}"
```

### 查询操作日志

```bash
curl http://localhost:8080/api/audit-logs \
  -H "Authorization: Bearer <token>"
```

## Docker Compose

```bash
docker compose up --build
```

Compose 中包含：

- 后端服务：`project-collaboration-platform`
- MySQL 8
- Redis 7
- Nginx 反向代理

## 后续扩展

- 接入 MySQL 与 MyBatis-Plus 持久化
- 使用 Redis 管理登录态、热点数据缓存和防重复提交
- 增加更细粒度的角色权限与菜单权限
- 增加单元测试、接口测试和 CI
- 补充前端管理页面
