# ECC 渐进式学习计划：从 0 到 1 搭建微服务解决方案

## 概述

本计划面向 **Java 后端工程师**，通过从零搭建一个微服务权限管理系统（参考 [RuoYi-Cloud-Plus](https://github.com/dromara/RuoYi-Cloud-Plus) 架构），在真实开发场景中逐步掌握 ECC（Everything Claude Code）的各项能力。

### 学习目标

- 掌握 ECC 核心概念：Agent、Skill、Command、Hook、Rule
- 在真实微服务开发中熟练使用 20+ 个 ECC 工具
- 从 0 到 1 完成一个可运行的企业级微服务系统
- 将 ECC 融入日常 Java 开发工作流

### 目标系统：MicroRbac 微服务权限管理系统

基于 RuoYi-Cloud-Plus 架构，构建简化版微服务 RBAC 系统：

```
micro-rbac/
├── micro-api/          # RPC 接口契约（Dubbo）
├── micro-common/       # 公共基础设施模块组
├── micro-gateway/      # API 网关（Spring Cloud Gateway）
├── micro-auth/         # 认证中心（Sa-Token + JWT）
├── micro-modules/      # 业务微服务
│   ├── micro-system/   # 系统管理（用户/角色/菜单/部门）
│   ├── micro-gen/      # 代码生成器
│   └── micro-job/      # 分布式任务调度
└── micro-visual/       # 监控中心（Spring Boot Admin）
```

### 技术栈

| 类别 | 技术 | 版本 |
|---|---|---|
| 基础 | Java + Spring Boot | 17 + 3.5.x |
| 微服务 | Spring Cloud | 2025.x |
| 注册/配置 | Nacos | 2.5.x |
| RPC | Apache Dubbo | 3.x |
| 网关 | Spring Cloud Gateway | — |
| 认证 | Sa-Token + JWT | 1.45.x |
| ORM | MyBatis-Plus | 3.5.x |
| 缓存 | Redisson | 3.52.x |
| 限流/熔断 | Sentinel | 1.8.x |
| 分布式事务 | Seata | 2.x |
| 任务调度 | SnailJob | 1.10.x |
| 链路追踪 | SkyWalking | 9.x |
| 监控 | Prometheus + Grafana + SBA | — |

---

## Phase 0: 环境准备与 ECC 基础概念

> **学习重点**: ECC 安装、核心架构概念、与 Claude Code 的协作模式

### 0.1 ECC 安装

```bash
# 在 Claude Code 中安装 ECC 插件
/plugin marketplace add https://github.com/affaan-m/ECC
/plugin install ecc@ecc
```

### 0.2 核心概念速览

ECC 是一个 Claude Code 的 Agent 操作系统，由 5 层架构组成：

```
┌──────────────────────────────────────────────────┐
│  Commands  │  斜杠命令入口（/plan, /code-review）   │
├──────────────────────────────────────────────────┤
│  Agents    │  AI 子代理（java-reviewer, architect）  │
├──────────────────────────────────────────────────┤
│  Skills    │  领域知识包（springboot-patterns）      │
├──────────────────────────────────────────────────┤
│  Hooks     │  自动化触发器（质量门禁、会话持久化）     │
├──────────────────────────────────────────────────┤
│  Rules     │  始终生效的规范（java/coding-style.md）  │
└──────────────────────────────────────────────────┘
```

| 概念 | 一句话理解 | Java 后端类比 |
|---|---|---|
| **Agent** | 可委派的 AI 子代理 | 微服务（各司其职） |
| **Skill** | 领域知识包 | Best Practice 文档 |
| **Command** | 用户调用的斜杠命令 | CLI 命令 |
| **Hook** | 事件驱动自动化 | AOP 切面 / Interceptor |
| **Rule** | 始终生效的规范 | Checkstyle / PMD 规则 |

### 0.3 Java 开发环境确认

确保以下环境已就绪：

```bash
java --version        # JDK 17+
mvn --version         # Maven 3.8+
docker --version      # Docker（用于 Nacos/Redis/MySQL）
```

### 0.4 第一个 ECC 命令：探索 Java 能力

```bash
# 查看 ECC 状态
/ecc:projects
```

在对话中输入以下内容，探索可用的 Java 工具：

```
ECC 中有哪些 Java/Spring Boot 相关的 skill 和 agent？请列出并简要说明
```

**你会看到**：ECC 列出 `springboot-patterns`、`springboot-tdd`、`springboot-security`、`java-coding-standards`、`jpa-patterns`、`java-reviewer`、`java-build-resolver` 等 15+ 个 Java 相关工具。

### 0.5 初始化项目

```bash
mkdir micro-rbac && cd micro-rbac
git init
```

在 Claude Code 对话中生成项目文档：

```
/init
```

**产出**: 项目根目录生成 `CLAUDE.md`，描述项目类型、技术栈和目录结构。

### Phase 0 验收

- [ ] ECC 安装完成，`/ecc:projects` 正常显示
- [ ] 能说出 Agent / Skill / Command / Hook / Rule 的区别
- [ ] 可列出 10+ 个 Java 相关的 skill 名称
- [ ] 项目 `CLAUDE.md` 已生成

---

## Phase 1: 项目初始化 — 多模块 Maven 工程搭建

> **学习重点**: `/ecc:plan` 规划、`architect` agent 架构设计、`springboot-patterns` skill

### 1.1 使用 `/plan` 规划项目结构

```
/ecc:plan 我要搭建一个微服务权限管理系统 micro-rbac，参考 RuoYi-Cloud-Plus 的架构。

模块划分：
- micro-api：RPC 接口契约（Dubbo Service 接口定义）
- micro-common：公共基础设施（core、redis、mybatis、security 等）
- micro-gateway：Spring Cloud Gateway 网关
- micro-auth：Sa-Token + JWT 认证中心
- micro-modules/micro-system：系统管理（用户、角色、菜单、部门）
- micro-visual/micro-monitor：Spring Boot Admin 监控

技术栈：JDK 17、Spring Boot 3.5.x、Spring Cloud 2025.x
请帮我制定详细的实现计划
```

**ECC 自动执行**：
1. 分析需求，生成分阶段计划
2. 识别风险（模块依赖、版本兼容性）
3. 列出文件清单和依赖关系
4. 等待你确认后开始执行

### 1.2 使用 `architect` agent 设计架构

```
用 architect agent 帮我设计 micro-rbac 的：
1. 模块依赖关系图
2. Maven 父子 POM 结构
3. 服务间通信方案（外部请求 → Gateway → REST → 各服务；内部调用 → Dubbo RPC）
4. 配置管理方案（Nacos 配置中心，多环境 profile）
```

### 1.3 创建父子 POM

```xml
<!-- 根 pom.xml -->
<groupId>com.example</groupId>
<artifactId>micro-rbac</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>

<modules>
    <module>micro-api</module>
    <module>micro-common</module>
    <module>micro-gateway</module>
    <module>micro-auth</module>
    <module>micro-modules</module>
    <module>micro-visual</module>
</modules>
```

### 1.4 ECC 工具调用示例

```
参考 springboot-patterns skill，确认我的多模块 Maven 项目结构是否合理
```

### 1.5 启动基础设施

```bash
# 启动 Nacos（服务注册 + 配置中心）
docker run -d --name nacos -p 8848:8848 -p 9848:9848 \
  -e MODE=standalone nacos/nacos-server:v2.5.1

# 启动 Redis
docker run -d --name redis -p 6379:6379 redis:7-alpine

# 启动 MySQL
docker run -d --name mysql -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 mysql:8.0
```

### Phase 1 验收

- [ ] Maven 多模块结构创建完成，`mvn compile` 通过
- [ ] `/ecc:plan` 生成的计划确认完毕
- [ ] `architect` agent 的架构设计已审查
- [ ] Nacos + Redis + MySQL 已启动

---

## Phase 2: 公共基础设施 — micro-common 模块

> **学习重点**: `java-coding-standards` skill、`java-reviewer` agent、`tdd-guide` agent

### 2.1 需要实现的模块

| 模块 | 功能 |
|---|---|
| `micro-common-core` | 基础工具类、异常体系、统一响应体 `R<T>` |
| `micro-common-redis` | Redisson 分布式缓存与锁封装 |
| `micro-common-mybatis` | MyBatis-Plus 封装（BaseEntity、分页、数据权限） |
| `micro-common-security` | 安全注解、权限校验工具 |
| `micro-common-web` | Web 层封装（全局异常处理、Jackson 序列化配置） |

### 2.2 实战：TDD 开发统一响应体

```
用 TDD 方式帮我实现 micro-common-core 的统一响应体 R<T> 类：
- code: int（状态码）
- msg: String（消息）
- data: T（数据）
- 静态工厂方法：ok(T data)、fail(int code, String msg)
- 使用 @Data 注解

先写测试，再写实现
```

ECC 工作流：
```
tdd-guide agent 规划测试
  → springboot-tdd skill 提供 JUnit 5 + AssertJ 示例
    → 先写 RTest.java（RED 阶段，测试失败）
      → 实现 R.java（GREEN 阶段，测试通过）
        → 重构优化
```

### 2.3 实战：封装 MyBatis-Plus 基础层

```
参考 jpa-patterns skill 的实体设计模式（适配 MyBatis-Plus），帮我设计：
1. BaseEntity（id, createTime, updateTime, createBy, updateBy，使用雪花ID）
2. BaseMapper<T> extends MyBatis-Plus BaseMapper
3. BaseService<T>（通用增删改查 + 分页查询）
4. 自动填充创建时间和更新时间

先写测试验证分页功能
```

### 2.4 代码审查

每写完一个模块后立即审查：

```
/ecc:code-review
```

或用 Java 专项审查：

```
用 java-reviewer agent 审查 micro-common-core 模块
```

**审查内容**：
- CRITICAL: SQL 注入风险、路径遍历、密钥硬编码
- HIGH: 异常处理完整性、事务边界
- MEDIUM: 命名规范、Optional 使用、Stream API

### Phase 2 验收

- [ ] 所有 common 子模块编译通过
- [ ] 单元测试覆盖率 ≥ 80%
- [ ] `java-reviewer` 审查无 CRITICAL 和 HIGH 问题
- [ ] 代码符合 `java-coding-standards` skill 规范

---

## Phase 3: 认证与网关 — micro-auth + micro-gateway

> **学习重点**: `springboot-security` skill、`api-design` skill、`security-reviewer` agent

### 3.1 micro-auth：认证中心

实现功能：

| 接口 | 说明 |
|---|---|
| `POST /auth/login` | 用户名密码登录，返回 JWT token + refreshToken |
| `POST /auth/register` | 用户注册 |
| `POST /auth/refresh` | 刷新 accessToken |
| `DELETE /auth/logout` | 登出，使 token 失效 |

```
/ecc:plan 实现 micro-auth 认证中心，技术栈：
- Sa-Token + JWT（无状态认证）
- 密码 BCrypt 加密（强度 10 轮）
- accessToken 有效期 2 小时，refreshToken 有效期 7 天
- 登录日志记录（IP地址、登录时间、结果）
- 登录失败次数限制（5 次锁定 15 分钟）

请参考 springboot-security skill 和 api-design skill 设计接口
```

### 3.2 安全审查

```
用 security-reviewer agent 审查 micro-auth 模块
```

安全审查重点：
- JWT secret 必须从环境变量读取 → 禁止硬编码
- 密码 BCrypt 强度 ≥ 10
- 登录失败限流（防暴力破解）
- 无认证信息泄露在日志中
- token 不过期前可被撤销

### 3.3 micro-gateway：API 网关

```
参考 springboot-patterns skill，实现 micro-gateway：
1. 路由转发（/system/** → micro-system, /auth/** → micro-auth）
2. 全局鉴权过滤器（从 Header 获取 JWT，调用 Sa-Token 校验）
3. 请求/响应日志记录（traceId 贯穿）
4. CORS 跨域配置
5. Sentinel 网关层限流（集成 sentinel-spring-cloud-gateway-adapter）
```

### 3.4 API 契约层

```
参考 api-design skill，在 micro-api 模块定义 Dubbo RPC 接口：
- RemoteUserService：根据 userId 查询用户信息（用于其他服务获取当前用户）
- RemoteLogService：记录操作日志（供各服务异步调用）

接口规范：
- 返回值统一用 R<T> 包装
- 参数做 Bean Validation 校验
- 接口命名遵循 {@code getByXxx / listByXxx / createXxx} 模式
```

### Phase 3 验收

- [ ] `/auth/login` 返回有效 JWT，gateway 可正确校验
- [ ] Gateway 路由转发正常，鉴权过滤器生效
- [ ] `security-reviewer` 审查无 CRITICAL 问题
- [ ] API 文档通过 SpringDoc 自动生成

---

## Phase 4: 核心业务服务 — micro-system 模块

> **学习重点**: `/ecc:orch-add-feature` 编排、`database-reviewer` agent、`api-design` skill

### 4.1 使用 `/orch-add-feature` 编排完整开发

```
/ecc:orch-add-feature 在 micro-system 模块实现用户管理功能：
- 用户 CRUD（分页查询、新增、修改、删除）
- 关联部门（多对一）和角色（多对多）
- 密码重置（仅管理员）
- 用户状态管理（启用/停用）
- 数据权限（普通用户仅看到本部门数据）
- @WebLog 操作日志自动记录
```

**`orch-add-feature` 编排流程**：

```
research（分析现有代码）
  → plan（生成实现计划）
    → tdd（先写测试，再写实现）
      → code-review（审查代码质量）
        → security-review（审查安全）
          → build-fix（修复构建问题）
            → pr（创建 Pull Request）
```

全程自动化，每步完成后才进入下一步。

### 4.2 分层架构实现

按照 `springboot-patterns` skill 的分层模式：

```
micro-system/src/main/java/com/example/system/
├── controller/
│   └── SysUserController.java         # @RestController + @Validated
├── service/
│   ├── ISysUserService.java           # 接口
│   └── impl/SysUserServiceImpl.java   # @Service + @Transactional
├── mapper/
│   └── SysUserMapper.java             # extends BaseMapper<SysUser>
├── domain/
│   ├── bo/SysUserBo.java              # 业务对象（入库用）
│   ├── vo/SysUserVo.java              # 视图对象（出参用）
│   └── convert/SysUserConvert.java    # MapStruct 转换接口
└── dubbo/
    └── RemoteUserServiceImpl.java     # @DubboService 暴露RPC
```

### 4.3 完整开发流程演示：用户管理

**步骤 1 — TDD 开发：**

```
用 TDD 方式开发 SysUserService.createUser：
- 用户名和邮箱不能重复
- 密码自动 BCrypt 加密存储
- 自动分配默认角色（ROLE_USER）
- 关联部门时校验部门是否存在
先写测试，确认失败后再写实现
```

**步骤 2 — 代码审查：**

```
/ecc:code-review
```

**步骤 3 — 数据库查询审查：**

```
用 database-reviewer agent 审查 SysUserMapper 和索引设计
```

**步骤 4 — 构建修复：**

```
/ecc:build-fix
```

**步骤 5 — 提交：**

```
/ecc:prp-commit 实现了系统管理模块的用户管理功能，
包含用户CRUD、角色关联、部门归属、密码加密，
单元测试覆盖率 87%
```

### 4.4 后续功能实现

以相同方式实现：

| 功能 | 关键复杂度 |
|---|---|
| 角色管理 | CRUD + 菜单权限树分配 + 数据范围设置 |
| 菜单管理 | 树形 CRUD + 按钮权限标识管理 + 图标选择 |
| 部门管理 | 树形 CRUD + 父子关系维护 + 数据权限范围 |

每次复用 `orch-add-feature` 编排流程。

### Phase 4 验收

- [ ] 用户/角色/菜单/部门 CRUD 全部完成
- [ ] Service 层单元测试覆盖率 ≥ 85%
- [ ] Controller 集成测试（MockMvc）通过
- [ ] `database-reviewer` 确认索引设计合理
- [ ] `java-reviewer` 审查通过

---

## Phase 5: 微服务治理 — 服务调用、熔断、事务

> **学习重点**: `backend-patterns` skill、多 Agent 并行审查、`database-migrations` skill

### 5.1 Dubbo RPC 远程调用

```
参考 backend-patterns skill 中微服务通信模式，实现跨服务调用：
1. micro-api 定义 RemoteUserService 接口
2. micro-system 端 @DubboService 暴露服务
3. micro-auth 端 @DubboReference 调用用户查询（登录时获取用户信息）
4. Dubbo 过滤器传递 Sa-Token 会话上下文（认证信息跨服务传播）
5. Dubbo 异常统一转换为 R<T> 格式
```

### 5.2 Sentinel 流量治理

```
集成 Sentinel 实现流量治理：
1. /system/user/** 接口配置 QPS 限流（每秒 100 次）
2. /auth/login 接口配置熔断规则（5 秒内失败 3 次 → 熔断 30 秒）
3. 自定义限流返回格式（统一使用 R<T> 包装，不为前端裸奔异常）
4. Docker 启动 Sentinel Dashboard
```

```bash
docker run -d --name sentinel -p 8718:8718 -p 8088:8080 \
  bladex/sentinel-dashboard:1.8.8
```

### 5.3 Seata 分布式事务

```
实现跨服务操作的事务一致性：
例如：创建用户时，micro-system 写用户表，同时 micro-resource 创建用户默认存储目录

1. Seata AT 模式配置（自动回滚）
2. @GlobalTransactional 注解标记事务边界
3. 测试正常提交和异常回滚
4. 用 database-migrations skill 管理两个库的 Flyway 迁移脚本
```

### 5.4 多 Agent 并行审查

修改了 RPC + 限流 + 事务代码后，并行审查（一次对话同时启动）：

```
同时运行以下审查：
1. java-reviewer agent 审查 micro-system 的 Dubbo 实现代码
2. security-reviewer agent 审查 RPC 调用中的会话传递安全性
3. database-reviewer agent 审查分布式事务涉及的数据库操作
```

### Phase 5 验收

- [ ] Dubbo RPC 调用正常，会话上下文正确传播
- [ ] Sentinel 限流和熔断规则生效
- [ ] Seata 分布式事务正确提交和回滚
- [ ] 所有 agent 审查通过

---

## Phase 6: 高级功能 — 代码生成、任务调度

> **学习重点**: `/ecc:multi-backend` 多模型工作流、`docker-patterns` skill

### 6.1 代码生成器 — micro-gen

```
参考 RuoYi-Cloud-Plus 的 ruoyi-gen 模块，实现代码生成器：
1. 查询 information_schema 获取表结构（字段名、类型、注释）
2. Velocity 模板引擎渲染模板（Controller/Domain/Mapper/Service/XML/Vue）
3. 支持选项：是否需要 CRUD/分页/导出/校验
4. API：GET /gen/listTables（查表列表）、POST /gen/generate（生成并下载）

用 /ecc:plan 规划实现方案
```

### 6.2 分布式任务调度 — micro-job

```
基于 SnailJob 实现 micro-job 模块：
1. 定时任务：每天凌晨 3:00 清理 90 天前的登录日志
2. 延迟任务：用户注册 24 小时后检查是否激活，未激活则发邮件提醒
3. 手动触发任务：管理员手动执行数据导出任务
4. SnailJob Dashboard 查看任务执行历史和状态
```

### 6.3 多模型性能优化

```
/ecc:multi-backend 优化 micro-system 的用户列表查询性能。
当前查询（关联角色表 + 部门表 + 岗位表）在 10万 用户数据下响应超过 1 秒，
请给出优化方案并实施
```

**`multi-backend` 并行分析**：
- 模型 A: SQL 层面优化（索引、JOIN 改写为多次查询 + 内存组装）
- 模型 B: 缓存策略（Redisson 缓存热点用户数据，Caffeine 本地缓存）
- 模型 C: 架构优化（读写分离、ES 搜索引擎替代复杂关联查询）
- 合并方案，选择实施

### Phase 6 验收

- [ ] 代码生成器可读取表结构并生成完整 CRUD 代码
- [ ] SnailJob 定时任务正常调度执行
- [ ] 用户列表查询优化后响应 < 200ms
- [ ] `multi-backend` 工作流完成并输出优化报告

---

## Phase 7: 监控运维 — 可观测性体系

> **学习重点**: `e2e-runner` agent、`/ecc:learn`、`docker-patterns` skill

### 7.1 监控中心 — micro-visual

```
搭建监控体系：
1. Spring Boot Admin — 服务健康监控、在线日志查看
2. Prometheus + Grafana — JVM 指标大屏（堆内存、GC、线程、CPU）
3. SkyWalking — 分布式链路追踪（请求经过的每一跳）
4. ELK — 日志集中收集（Logstash → Elasticsearch → Kibana）

用 docker-patterns skill 管理 docker-compose 编排
```

### 7.2 E2E 端到端测试

```
用 e2e-runner agent 生成关键流程的端到端测试：
1. 注册用户 → 登录获取 token → 创建角色 → 分配给用户 → 验证权限生效
2. 未登录访问接口 → 验证返回 401
3. 无权限访问接口 → 验证返回 403
4. 并发登录 → 验证限流规则生效
```

### 7.3 提取学习成果

```
/ecc:learn 从 Phase 0 到 Phase 7 的整个开发过程中提取可复用的模式
```

Continuous Learning 系统会从 session 历史中提取高质量模式，保存为 instinct，后续开发中自动激活。

### Phase 7 验收

- [ ] Spring Boot Admin 可查看所有服务健康状态
- [ ] Grafana 大屏显示 JVM 运行指标
- [ ] SkyWalking UI 可追踪完整调用链
- [ ] E2E 测试覆盖核心认证 + 权限流程
- [ ] `/ecc:learn` 成功提取新的 instinct

---

## Phase 8: 部署与 CI/CD

> **学习重点**: `deployment-patterns` skill、`springboot-verification` skill

### 8.1 Docker 容器化

```
用 deployment-patterns skill 完成容器化部署：
1. 为每个微服务编写 Dockerfile（多阶段构建减小镜像体积）
2. docker-compose.yml 编排所有服务（Nacos + MySQL + Redis + 各微服务）
3. 支持多环境配置切换（dev/staging/prod）
```

### 8.2 完整构建验证

```
用 springboot-verification skill 对 micro-rbac 做完整构建验证
```

验证流程：
```bash
mvn clean verify          # 编译 + 测试
mvn jacoco:report         # 覆盖率报告
mvn spotless:check        # 代码格式检查
```

### 8.3 创建 Pull Request

```
/ecc:pr
```

自动分析 `git log`，生成 PR 标题和描述，包含完整的测试计划 checklist。

### Phase 8 验收

- [ ] `docker-compose up` 一键启动所有服务
- [ ] 整体测试覆盖率 ≥ 80%
- [ ] PR 已创建，CI 检查全部通过

---

## ECC 学习路径总览

```
Phase 0    ██ 安装 + 概念
           工具: /init, /ecc:projects
           ┃
Phase 1    ████ 多模块工程
           工具: /ecc:plan, architect agent, springboot-patterns skill
           ┃
Phase 2    ██████ 公共基础设施
           工具: tdd-guide agent, java-reviewer, java-coding-standards skill
           ┃
Phase 3    ██████ 认证网关
           工具: security-reviewer, springboot-security skill, api-design skill
           ┃
Phase 4    ██████████ 核心业务（最核心）
           工具: /orch-add-feature, database-reviewer, /code-review
           ┃
Phase 5    ██████ 微服务治理
           工具: backend-patterns skill, 多Agent并行审查, database-migrations
           ┃
Phase 6    ██████ 高级功能
           工具: /multi-backend, /ecc:plan
           ┃
Phase 7    ████ 监控运维
           工具: e2e-runner, /ecc:learn, docker-patterns skill
           ┃
Phase 8    ████ 部署CI/CD
           工具: deployment-patterns skill, springboot-verification, /ecc:pr
```

---

## Java 微服务 ECC 速查表

### 开发阶段

| 场景 | ECC 工具 | 用法 |
|---|---|---|
| 功能规划 | `/ecc:plan` | 对话中直接调用 |
| 架构设计 | `architect` agent | "用 architect agent 设计..." |
| TDD 开发 | `tdd-guide` agent + `springboot-tdd` skill | "用 TDD 方式实现..." |
| 完整功能编排 | `/ecc:orch-add-feature` | 对话中直接调用 |
| 多模型优化 | `/ecc:multi-backend` | 对话中直接调用 |

### 代码质量

| 场景 | ECC 工具 | 用法 |
|---|---|---|
| Java 审查 | `java-reviewer` agent | "用 java-reviewer 审查模块" |
| 通用审查 | `/ecc:code-review` | 对话中直接调用 |
| 安全审查 | `security-reviewer` agent | "用 security-reviewer 审查" |
| 数据库审查 | `database-reviewer` agent | "审查索引和查询" |
| 构建修复 | `/ecc:build-fix` | 编译报错时调用 |
| 代码简化 | `code-simplifier` agent | "简化这段代码" |

### 领域知识 Skill

| 场景 | Skill | 激活方式 |
|---|---|---|
| Spring Boot 架构 | `springboot-patterns` | "参考 springboot-patterns..." |
| 安全认证 | `springboot-security` | "参考 springboot-security..." |
| 数据层 | `jpa-patterns` | "参考 jpa-patterns..." |
| REST API | `api-design` | "参考 api-design..." |
| 数据库迁移 | `database-migrations` | "参考 database-migrations..." |
| Redis 缓存 | `redis-patterns` | "参考 redis-patterns..." |
| Docker 部署 | `docker-patterns` | "参考 docker-patterns..." |
| 后端通用模式 | `backend-patterns` | "参考 backend-patterns..." |
| Java 编码规范 | `java-coding-standards` | 自动通过 Rule 加载 |

### Git 协作

| 场景 | ECC 工具 | 用法 |
|---|---|---|
| 快速提交 | `/ecc:prp-commit` | `/ecc:prp-commit 变更描述` |
| 创建 PR | `/ecc:pr` | 对话中直接调用 |
| 审查 PR | `/ecc:review-pr` | 对话中直接调用 |

---

## 学习建议

### 时间安排

| 阶段 | 建议时长 | 难度 | 关键产出 |
|---|---|---|---|
| Phase 0 | 0.5 天 | 低 | ECC 安装，理解基本概念 |
| Phase 1 | 1 天 | 中 | Maven 多模块工程可编译 |
| Phase 2 | 2-3 天 | 中 | common 模块完成，覆盖率达标 |
| Phase 3 | 2-3 天 | 高 | 认证网关可运行，安全审查通过 |
| **Phase 4** | **4-5 天** | **高** | **核心 CRUD 全部完成（最重要）** |
| Phase 5 | 2-3 天 | 高 | 微服务治理三大组件集成 |
| Phase 6 | 2-3 天 | 中 | 代码生成器 + 任务调度 |
| Phase 7 | 1-2 天 | 中 | 监控大屏 + 链路追踪 |
| Phase 8 | 1-2 天 | 中 | 一键部署 + CI/CD |

**总计：约 3-4 周完成全部阶段。**

### 学习方法

1. **先理解目标再动手** — 每个 Phase 先看 ECC 工具能做什么，再实际使用
2. **构建错误用 `/build-fix`** — 不手动排查，让 AI 自动修复
3. **修改后立即审查** — `/code-review` 形成肌肉记忆
4. **坚持 TDD** — Phase 2 到 Phase 6 所有新增代码都用 TDD
5. **Phase 4 投入最多时间** — 这是核心，后面越来越快

### 进阶方向

完成本计划后，继续探索：
- `/ecc:orch-build-mvp` — 从设计文档直接生成可工作的 MVP
- `/ecc:epic-decompose` — 自动拆解大型需求
- `/ecc:rules-distill` — 从 skill 中提取跨领域规则
- `continuous-learning-v2` skill — 打造个人知识积累系统
- `autonomous-loops` skill — 配置自主 Agent 循环（自动监控 CI 失败并修复）
- `/ecc:council` — 多角色 AI 会诊复杂决策

---

> **文档版本**: v1.0
> **创建日期**: 2026-07-22
> **基于**: ECC v2.0.0 + RuoYi-Cloud-Plus v2.6.2
> **适用人群**: Java 后端工程师（需 Spring Boot 基础，了解微服务概念）