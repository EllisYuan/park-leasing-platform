# 01: 可运行登录与Session闭环

Status: open
Label: ready-for-agent
Type: task
Blocked by: None (can start immediately)
Spec: [平台Spec](../spec.md)
User stories: 4, 5

## What to build

从浏览器真实登录进入应用：账号密码认证建立 Spring Security 服务器 Session，经 HttpOnly Cookie 载入身份，前端查询当前身份并进入已登录界面；退出销毁当前会话；闲置 30 分钟需重新登录。此票不是只创建空脚手架，必须同时交付可运行的前后端、MySQL migration 与 HTTP/API 测试 seam，并在实际构建与启动中核对 Java 21、受支持的 Spring Boot 稳定线、MyBatis-Plus、Spring Security、Flyway、springdoc-openapi 与 Vue 3 工具组合，记录锁定版本与依赖树。

认证按服务器 Session、HttpOnly Cookie 与 CSRF 实现，开发前端经代理、部署同源；单实例内存会话不持久化，重启后须重新登录，允许多设备同时在线。CSRF 防护覆盖状态改变请求。演示管理员通过安全初始化创建，不编造通用密码或伪造 hash，也不在文档中写入未实际产生的初始化结果。

- 登录页提交账号密码后进入应用壳；前端以身份查询恢复会话状态，未认证时回到登录页。
- 退出只销毁当前会话，其他设备会话不受影响；服务端判定超时，前端收到未认证后要求重新登录。
- 状态改变请求统一携带 CSRF token，缺失或错误时返回一致拒绝。
- 初始化管理员凭据只经安全初始化过程产生，交付记录初始化方式，不写入通用密码或虚构结果。
- 最小 API 文档与统一错误返回结构作为后续票据的测试与契约基础。
- MySQL migration 与测试数据库生命周期在实现中锁定，测试可重复执行。

## Acceptance criteria

- [ ] 空 MySQL 上执行 Flyway migration 后应用可直接启动，身份与账号表存在，springdoc 暴露登录与身份查询的 API 文档。
- [ ] 浏览器以初始化管理员登录成功后，身份查询返回当前用户并进入已登录界面；登出后同一查询返回未认证，需重新登录。
- [ ] 未携带 CSRF token 的状态改变请求被拒绝，携带合法 token 的同一请求成功。
- [ ] Session Cookie 为 HttpOnly，页面 JavaScript 读取不到；HTTPS 配置下带 Secure 属性。
- [ ] 闲置超时在可控时间推进下使会话失效，下一次请求要求重新登录，测试不使用真实等待。
- [ ] 重启后端后原会话失效，重新登录可继续使用；两个浏览器会话可同时在线。
- [ ] 错误密码返回不区分账号是否存在的统一失败信息，不泄露账号枚举线索。
- [ ] HTTP/API 测试连接真实 MySQL 覆盖登录、身份查询、登出与超时，不使用替代数据库，测试原始输出作为证据保留。
- [ ] 实际构建与启动命令可重复执行，锁定的版本组合与依赖树已记录；未实际运行的项保持未验证表述。

## Context

父级范围见 [平台Spec](../spec.md)，身份与认证条目在 Implementation Decisions 第 8—9 条，验收方向在 Testing Decisions 的“身份授权”与“主测试 seam”。术语按 [术语表](../../../GLOSSARY.md) 使用“园区角色授权”“企业账号”等原词。

认证方案见 [ADR 0003](../../../docs/adr/0003-browser-session-authentication.md)，同仓库模块化单体与工程结构见 [ADR 0001](../../../docs/adr/0001-modular-monolith.md)。本票只建立最小可运行的认证与工程 seam，员工与授权管理在 02 展开，后续业务模块复用此会话与测试边界。
