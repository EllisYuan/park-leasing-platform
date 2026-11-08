# 园区厂房租赁与招商管理平台

面向园区运营商，规划招商门户、企业端和运营端，共用后端与数据库。

业务主链：厂房建档 → 企业线索与跟进 → 看房 → 报价意向与审批占房 → 合同与签署 → 交付 → 分段出账与收款 → 续租或退租交还 → 押金结算。

## 当前状态

已初始化本地 Git 仓库与 agent 协作规则。当前尚无业务应用、数据库 migration 或运行验证；已有 `plugins/` 是 agent 工具。

已确认 Java 21、Maven、Spring Boot（优先评估4系列受支持稳定线）、MyBatis-Plus、MySQL 8.4，以及 Vue 3、TypeScript、Vite、Element Plus、Pinia。认证采用 Spring Security Session + HttpOnly Cookie + CSRF。具体版本与依赖组合待最小集成验证，尚未锁定。

## 文档入口

- [正式 Spec](.scratch/park-leasing-platform/spec.md)：整合Q1—Q27、110条用户故事、实现与测试决定；本地标记ready-for-agent，尚未启动实现。
- [Tickets 索引](.scratch/park-leasing-platform/tickets.md)：用户批准的28项vertical slices、直接依赖及领取规则。
- [当前设计基线](docs/design/platform-baseline.md)：Q1—Q27确认的范围、技术方向、结构和交付，待最终整体确认。
- [核心业务规则](docs/design/core-business-rules.md)：招商、合同、计费、收款、押金、工单与统计。
- [数据与接口设计](docs/design/data-api-design.md)：拟定实体、API与事务边界，尚非实际schema或已实现API。
- [实施与交付](docs/design/delivery-plan.md)：阶段、界面导航、部署和验收。
- [领域术语](GLOSSARY.md)与 [ADR](docs/adr/)：已确认术语及重要取舍。
- [本地讨论记录](.scratch/platform-design/map.md)：逐轮确认与待定事项。
- [上级项目导航](../explore/README.md)：前期业务范围、模块、数据模型和技术建议；与本轮确认的差异待统一。
- [项目规则](CLAUDE.md)：按任务读取材料与证据边界。
- [Issue tracker](docs/agents/issue-tracker.md)：Local markdown，任务保存在 `.scratch/<feature>/`。
- [Domain docs](docs/agents/domain.md)：single-context 的术语和 ADR 读取规则。

上级 `explore/` 位于本 Git 仓库之外，单独 clone 本仓库时需要另行取得规划材料。设计 SQL 为备选方案，尚未作为数据库 migration 执行。

## 开发与验证

业务构建清单、启动命令和测试命令尚未建立，创建应用时以实际配置补充。验证记录区分已执行、失败和未执行，论文与演示材料引用实际运行证据。
