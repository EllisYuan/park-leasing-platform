# 工程实现规范（所有 ticket 必须遵循）

本文是 28 张 ticket 的共同工程契约。实现细节由各 ticket 决定，但目录、技术版本、API 形状、权限、数据库、测试与证据方式必须一致。若本文与实际代码冲突，以 `integration/park-leasing-platform` 上已合入的代码为准，并在该 ticket 的 Answer 中记录偏差。

## 0. 已验证环境与版本（2026-11-08）

最小集成 spike 已实际构建、启动并通过 Testcontainers 测试：

| 组件 | 版本 |
| --- | --- |
| Java | 21（`JAVA_HOME=C:\Java\jdk-21.0.9`；PATH 上的 `java` 可能是 17，必须用 Maven/JAVA_HOME） |
| Maven | 3.9.12 |
| Spring Boot | 4.1.2 |
| MyBatis-Plus | 3.5.18（`mybatis-plus-spring-boot4-starter`） |
| Spring Security | 7（由 Boot 管理） |
| Flyway | Boot 4 模块：`spring-boot-starter-flyway` + `org.flywaydb:flyway-mysql` |
| MySQL | 8.4（Testcontainers `mysql:8.4`，镜像已在本地） |
| springdoc-openapi | 3.0.2（`springdoc-openapi-starter-webmvc-ui`） |
| Testcontainers | 2.0.5（artifact 名为 `testcontainers-mysql`、`testcontainers-junit-jupiter`；`MySQLContainer` 非泛型，包 `org.testcontainers.mysql`） |
| Node / npm | 24.11.1 / 11.6.2 |
| 前端 | Vue 3 + TypeScript + Vite + Vue Router + Pinia + Element Plus（首次安装后锁定 `package-lock.json`） |

环境注意：

- Maven Central 不可达，必须走阿里云镜像：`server/.mvn/settings.xml` + `server/.mvn/maven.config`（内容 `-s .mvn/settings.xml`）。始终 `cd server` 后执行 `mvn`。
- npm registry 已全局指向 `https://registry.npmmirror.com`；可在 `web/.npmrc` 显式声明。
- Windows 上运行中的 jar 会锁定文件：重新构建前先停止应用（不要全局 kill java，按端口定位自己的进程）。
- 集成测试统一使用 Testcontainers 启动 MySQL 8.4，不依赖本机 MySQL、不持久化任何数据库口令。Docker Desktop 必须运行。

## 1. 仓库结构

```
server/                     Spring Boot 单应用（Maven）
web/                        Vue 3 单 SPA
deploy/                     Nginx、Compose、备份恢复脚本
docs/engineering-conventions.md
docs/evidence/<NN>-<slug>/  每个 ticket 的真实输入输出证据
scripts/                    开发辅助脚本
.scratch/                   issue tracker（spec、tickets）
```

后端包根：`com.parkleasing`。按业务分包，包内按需 `controller/service/mapper/entity/dto/vo`：

```
common/       统一响应、错误、分页、审计、Clock、ID/金额序列化
security/     认证、会话、CSRF、权限注解、数据范围解析
system/       员工、部门、园区、角色授权、字典、公告、待办、审批记录
asset/        园区、楼栋、出租单元、价格与可用性
enterprise/   承租企业、联系人、账号归属、成员权限、资料变更
investment/   线索、跟进、带看、意向、预约、公海
lease/        合同、审批轮次、签署、交接、续租、终止、结算
finance/      账单、出账、收款、押金、调整、红冲、退款、转结
property/     报修工单
reporting/    只读统计与导出
file/         私有文件服务
portal/       公开门户接口（只读公开数据）
```

规则：

- 跨业务协作只能调用对方公开的 `Service` 方法；事务由用例入口 Service 持有，禁止在事务内做内部 HTTP 调用。
- `reporting` 只读，不维护第二套业务事实。
- 禁止在 controller 中写业务规则；controller 只做参数绑定与调用。
- 不引入 Redis、消息队列、WebSocket、外部支付。

## 2. 构建与运行

后端：

```bash
cd server
mvn -q test                 # 全部测试（Testcontainers）
mvn -q -DskipTests package  # 生成可执行 jar
```

开发运行使用 `scripts/run-dev.sh`（ticket 01 建立）：脚本在运行时生成临时数据库口令并启动 MySQL 容器，把连接信息通过环境变量传给应用；不得把口令写入仓库或文档。

前端：

```bash
cd web
npm ci          # 首次可用 npm install 生成 lockfile
npm run build
npm run typecheck
npm run dev     # Vite 代理 /api -> http://localhost:8080
```

## 3. 数据库与 migration

- 表名、字段名沿用 `docs/design/data-api-design.md` 的拟定名称（`park`、`building`、`workshop_unit`、`enterprise`、`customer_lead`、`intention_order`、`contract`、`contract_unit`、`lease_handover`、`bill`、`bill_item`、`payment_record`、`deposit_record`、`deposit_transaction`、`lease_settlement`、`attachment`、`repair_order`、`sys_user` 等），ticket 可扩展但不得无故改名。
- migration 文件：`server/src/main/resources/db/migration/V<NN><SS>__<slug>.sql`。`NN` 为 ticket 编号两位（01—28），`SS` 为该票内序号。例：`V0101__auth_and_account.sql`。**并行 ticket 不得占用其他票的 NN 前缀。**
- 已合入的 migration 不得修改；变更追加新文件。
- 所有表 InnoDB、`utf8mb4_0900_ai_ci`。主键 `BIGINT AUTO_INCREMENT`。
- 审计列：`created_at TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3)`、`created_by BIGINT NULL`、`updated_at TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3)`、`updated_by BIGINT NULL`。
- 外键显式命名 `fk_<table>_<target>`；唯一键 `uk_...`；普通索引 `idx_...`。
- 业务历史（合同、账单、资金、审批、交接、工单轨迹）不做物理删除；用状态列表达。
- 金额 `DECIMAL(18,2)`（台账/账单/收款），单价 `DECIMAL(18,4)`；百分比/系数 `DECIMAL(9,6)`。
- 业务日期 `DATE`；审计时间 `TIMESTAMP(3)`（UTC，Java 用 `Instant`）。
- 区间冲突用数据库锁与条件校验实现，普通唯一键不足以表达日期区间互斥。

## 4. API 契约

统一前缀 `/api`（公开门户 `/api/portal/**`，匿名可访问）。

- 成功响应：HTTP 200（查询/命令）或 201（创建），body：

```json
{ "code": "OK", "message": "ok", "data": { } }
```

- 错误响应：HTTP 状态与业务码一致，body：

```json
{ "code": "STABLE_BUSINESS_CODE", "message": "面向用户的中文说明", "data": null, "traceId": "..." }
```

HTTP 映射：400 参数/业务校验；401 未认证；403 已认证但无权；404 不存在或不可见（跨租户一律 404，避免存在性泄露）；409 状态冲突/并发冲突/重复；500 未预期错误（不返回堆栈）。前端不得根据 message 文本分支。

- 分页：query `page`（1 起）、`size`（默认 20，最大 100）；响应 `{ "items": [...], "total": 123, "page": 1, "size": 20 }`。
- ID：JSON 中 BIGINT 一律序列化为字符串。
- 金额：JSON 中一律十进制字符串（`"1234.50"`），后端 `BigDecimal`，使用 `ToStringSerializer` 与 `WRITE_BIGDECIMAL_AS_PLAIN`。
- 日期：`LocalDate` → `yyyy-MM-dd`；审计时间 `Instant` → ISO-8601 带 `Z`（UTC）。
- 状态改变命令使用 POST，不通过普通 DELETE 抹除历史。
- OpenAPI：`/v3/api-docs`、`/swagger-ui.html` 可匿名访问（仅文档，不含业务数据）。

## 5. 认证、会话与 CSRF（ticket 01 建立）

- Spring Security 服务器 Session；`JSESSIONID` 必须 HttpOnly；HTTPS profile 下 Secure；SameSite=Lax。
- 闲置 30 分钟超时（`server.servlet.session.timeout=30m`），无长期免登录，多设备可同时在线，单实例内存会话，重启需重新登录。
- CSRF：`CookieCsrfTokenRepository`，cookie 名 `XSRF-TOKEN`（非 HttpOnly，供 SPA 读取），请求头 `X-XSRF-TOKEN`。提供 `GET /api/auth/csrf` 触发 token 下发。登录/退出等状态改变请求必须校验 CSRF。
- 端点：`POST /api/auth/login`、`POST /api/auth/logout`、`GET /api/auth/me`、`GET /api/auth/csrf`。
- 登录失败统一返回 `AUTH_INVALID_CREDENTIALS` 与相同 message，不区分账号是否存在、是否停用。
- 密码 BCrypt。初始管理员只由安全初始化创建：读取环境变量 `PLP_INITIAL_ADMIN_USERNAME` / `PLP_INITIAL_ADMIN_PASSWORD`，缺失则不创建并记录警告；不得写死通用密码或伪造成功输出。
- 账号停用、密码修改、园区角色调整后，使该用户已有会话在下一次请求失效（`SessionRegistry` + `HttpSessionEventPublisher`；按 principal 过期全部会话）。
- `GET /api/auth/me` 返回：`id`、`username`、`displayName`、`userType`（`STAFF`/`ENTERPRISE`）、`systemAdmin`、`enterpriseId`、`parkRoles`（`[{parkId, parkName, roleCode}]`）、`permissions`（字符串数组）。前端只据此渲染入口。

## 6. 权限与数据范围（ticket 02 建立，所有票复用）

- 角色：`SYSTEM_ADMIN`（全局，不自动获得园区经营数据）、`PARK_MANAGER`、`LEASING_MANAGER`、`LEASING_AGENT`、`CONTRACT_SPECIALIST`、`FINANCE`、`PROPERTY`。
- 授权关系表：`staff_park_role(staff_id, park_id, role_code)`，同一员工可跨园区多条。
- 权限字符串（示例，按票扩展）：`asset:read`、`asset:write`、`lead:read`、`lead:write`、`lead:assign`、`intent:submit`、`intent:approve:manager`、`intent:approve:park`、`contract:submit`、`contract:finance`、`contract:final`、`finance:billing`、`finance:receipt`、`finance:deposit`、`finance:refund:approve`、`finance:refund:execute`、`finance:reversal:approve`、`property:handover`、`property:repair`、`report:read`、`export:ledger`。
- 服务端统一 `ScopeService`：解析当前用户的授权园区集合、部门/本人范围、企业归属。所有列表、详情、统计、导出、附件、状态命令都必须经过它；不得信任前端提交的 `enterpriseId`/`parkId` 归属。
- 系统管理员没有园区角色时，业务数据接口返回 403；管理账号/园区/角色是独立能力，不等于经营数据访问。
- 提交人与审核人必须不同：`ApprovalGuard.requireDifferentActor(submitterId, approverId)`，所有审批命令复用。
- 跨企业访问：一律 404；跨园区无权：403；列表按授权园区过滤。

## 7. 常用技术约定

- `Clock` bean：`Clock.system(ZoneId.of("Asia/Shanghai"))`；所有“当前时间/今天”从注入的 `Clock` 获取，禁止直接用 `LocalDate.now()`/`Instant.now()`。测试用可变 Clock。
- 业务时区常量 `Asia/Shanghai`；审计时间存 UTC。
- 幂等：业务号/请求号 + 数据库唯一约束；重复请求返回既有结果，不重复写流水。
- 锁顺序：按全局文档固定（单元 ID 升序；账单→收款；押金账户→申请→账单）；更新状态带旧状态/版本条件。
- 并发测试必须使用真实 MySQL 多线程/多连接，不能只 mock。
- 日志不输出密码、Cookie、完整凭证文件内容。

## 8. 私有文件服务契约（ticket 04 建立）

- 表 `attachment`：`id, storage_key, original_name, content_type, size_bytes, sha256, visibility(PRIVATE/PUBLIC), status(TEMP/BOUND), owner_type, owner_id, uploaded_by, created_at, bound_at`。
- `POST /api/attachments`（multipart）→ 返回 attachment id，状态 `TEMP`；单文件 ≤ 20MB；白名单 `image/jpeg,image/png,image/webp,application/pdf`；服务端校验 content type 与大小。
- `POST /api/attachments/{id}/bind`（`ownerType`, `ownerId`）→ 服务端校验调用者对该业务对象的权限后置 `BOUND`。
- `GET /api/attachments/{id}/download` → 认证 + 业务归属鉴权；`PUBLIC` 且业务允许公开的图片可匿名访问；否则 404。
- 物理存储根 `plp.file.root`（默认 `./data/files`），随机 storage key；绝不暴露路径、不做静态目录映射。
- 未绑定 `TEMP` 文件由定时任务清理（默认 24h），记录清理结果。
- 数据库事务不能回滚文件写入：先写临时文件再落库；失败路径删除孤儿文件。

## 9. 前端约定

- 一个 Vue SPA，三布局：`PortalLayout`（`/`、`/parks/:id`、`/units/:id`）、`EnterpriseLayout`（`/enterprise/*`）、`AdminLayout`（`/admin/*`），`/login` 独立。
- 路由守卫：未登录访问受保护页跳 `/login`；已登录按 `me.permissions` 渲染菜单。
- `src/api/http.ts`：`fetch` + `credentials: 'include'`；自动带 `X-XSRF-TOKEN`；401 跳登录；统一解析 `{code,message,data}`；错误 `ElMessage` 提示。
- Element Plus 组件为主；浅色中性背景 + 深蓝重点色；后台台账/审批密集，门户重展示，企业端重待办与账单。
- 门户与企业端在 375px 宽度可用；后台 PC 优先、手机基本可用。
- 金额显示保留两位小数并带千分位；日期 `YYYY-MM-DD`。
- 所有列表页具备分页、筛选、空状态与错误提示。

## 10. 测试约定

- 后端集成测试：`@IntegrationTest` 元注解（ticket 01 建立），包含：

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@Import(TestcontainersConfiguration.class)
```

`TestcontainersConfiguration` 提供单例 `MySQLContainer("mysql:8.4")` + `@ServiceConnection`，Flyway 自动迁移。

- 真实 HTTP：测试使用 JDK `HttpClient` 封装 `ApiClient`（维护 Cookie、自动登录、自动带 CSRF、`get/post/put/delete`，返回状态码+解析后的 body）。不使用 MockMvc 作为唯一证据；关键权限/并发/资金用例必须走真实端口。
- 每个测试前清理业务表：`DatabaseCleaner` 禁用外键检查后 TRUNCATE 所有非 `flyway_schema_history` 表。测试自建所需账号/园区/企业。
- 时间：`MutableClock`（可推进的 `Clock`）通过 `@TestConfiguration @Primary` 注入；超时/到期测试不得真实等待。
- 金额 oracle：计费测试用独立手算基准（不复制实现算法）构造预期值。
- 并发：用 `ExecutorService` + 多线程 HTTP 请求验证唯一约束/行锁；断言只有一个成功。
- 前端：每个 ticket 至少 `npm run build` 与 `npm run typecheck` 通过；关键交互可用 Playwright（若安装）或记录手工验证命令与截图来源。不得把未执行的浏览器验证写为通过。
- 测试输出保留原始日志；不得伪造通过结论。

## 11. 证据与 ticket 收尾

每个 ticket 在 `docs/evidence/<NN>-<slug>/` 保存：

- `README.md`：执行命令、日期、环境、结果摘要、未执行项。
- `*.log` 或 `*.txt`：真实命令输出（可裁剪重复行，但保留关键断言与统计）。
- 前端截图（若执行）：注明页面、宽度与运行方式。

ticket 文件（`.scratch/park-leasing-platform/issues/NN-*.md`）：

1. 开始时 `Status: open` → `Status: claimed`。
2. 完成后在文件底部追加 `## Answer`，写明实现范围、关键文件、验证命令与结果、已知缺口；`Status: resolved`。
3. 勾选已满足的 Acceptance criteria（未满足保持未勾选并说明）。

Git：

- 每个 ticket 在独立分支 `ticket/NN-<slug>`、独立 worktree 上开发。
- 提交信息中文，不写模型信息。
- 完成前把 `integration/park-leasing-platform` 最新提交 merge 进本分支并重跑受影响测试。

## 12. 已知限制

- 本期不实现在线支付、真实打款、电子签章、短信/邮件/WebSocket、能耗分摊、政策申报、3D。
- 部署为单实例；会话不持久化；未配置 HTTPS 时不声称 Secure Cookie 已启用。
- 具体性能数字、P95、可用率不得预填；只记录实际测量。
