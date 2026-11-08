# Domain docs

## 探索前读取

采用 single-context：

- 根目录 `GLOSSARY.md`：领域术语。
- `docs/adr/`：读取与本次任务有关的 ADR。

文件尚不存在时继续任务；在术语或架构决定需要落文档时，由 `domain-modeling` skill 按需创建。

## 使用规则

- Issue、设计、代码与测试使用 `GLOSSARY.md` 定义的领域词汇。
- 缺少必要术语时记录缺口，交由 `domain-modeling` 处理。
- 提议与已有 ADR 冲突时，明确引用该 ADR 并说明冲突。
- 上级业务方案入口为 [项目导航](../../../explore/README.md)；其中方案状态与实际实现分别核对。
