# 园区厂房租赁与招商管理平台

当前处于仓库初始化阶段。已有 `plugins/` 为 agent 工具，不代表业务实现。

## 项目入口

- 需求、业务范围与模块：先读 `../explore/README.md`，再按任务读取其指向的文档。
- 开发与架构：读取 `../explore/04-技术方案与架构.md`；版本在最小集成验证后锁定。
- 数据库与 API：读取 `../explore/03-数据模型设计.md` 和相关设计 SQL，再核对实际源码与 migration。
- 验收与论文：读取 `../explore/06-实施路线与论文对齐.md`；规划完成边界见 `../explore/09-修订与交付核验.md`。

设计文档描述预期方案；实现事实以当前源码和实际验证结果为准。交付时说明修改位置、已执行验证与剩余缺口。

## Agent skills

### Issue tracker

任务和 spec 使用本仓库 `.scratch/` 下的 Local markdown。规则见 `docs/agents/issue-tracker.md`。

### Domain docs

采用 single-context：根目录 `GLOSSARY.md` 与 `docs/adr/`。读取规则见 `docs/agents/domain.md`。
