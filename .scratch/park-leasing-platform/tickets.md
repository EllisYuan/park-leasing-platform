# 平台实施 Tickets

依据用户明确批准的28项拆分，父级为 [平台Spec](spec.md)。本轮只发布Local Markdown任务，未授权开始代码实现或提交Git。每项是可验证的vertical slice，数据库变更、API、UI、权限及必要并发/金额/幂等测试随该项完成。

## 状态与领取

各ticket使用 `Status: open`、`Label: ready-for-agent`、`Type: task`。Label表示描述已可供agent使用，不表示忽略依赖即可领取。只有 `Status: open` 且 `Blocked by` 全部为 `resolved` 的ticket属于当前ready frontier；领取时先记 `claimed`，验证完成后追加Answer及证据并记 `resolved`。本轮不领取或关闭任务。

依赖只列直接阻塞，省略传递依赖；编号不是强制串行执行顺序。最初只有01可开始；02完成后03/04/05可并行。跨票共享能力以已合入结果复用，代码文件冲突不通过虚构业务依赖解决。

## 任务清单

| Ticket | 独立文件 | Blocked by |
| --- | --- | --- |
| 01 可运行登录与Session闭环 | [01](issues/01-runtime-session-login.md) | 无 |
| 02 员工、部门与园区角色授权 | [02](issues/02-staff-park-role-scope.md) | 01 |
| 03 楼栋、出租单元与基础租控 | [03](issues/03-workshop-units-availability.md) | 02 |
| 04 企业核验开户与私有资料 | [04](issues/04-enterprise-onboarding-private-files.md) | 02 |
| 05 字典、公告与站内待办 | [05](issues/05-dictionary-notices-tasks.md) | 02 |
| 06 招商线索、跟进与带看记录 | [06](issues/06-leads-follow-ups-viewings.md) | 03 |
| 07 报价审批与7天意向暂占 | [07](issues/07-intent-approval-seven-day-hold.md) | 05、06 |
| 08 多单元合同审批与签署 | [08](issues/08-multi-unit-contract-approval-signing.md) | 04、07 |
| 09 IN/OUT交接与到期占用管理 | [09](issues/09-handover-expiry-physical-occupation.md) | 08 |
| 10 自然月分段账单与补漏 | [10](issues/10-segmented-billing-catch-up.md) | 08 |
| 11 线下部分收款与企业查询 | [11](issues/11-offline-partial-receipts-enterprise-read.md) | 10 |
| 12 押金收取与余额流水 | [12](issues/12-deposit-receipts-ledger.md) | 11 |
| 13 账单作废重开与关联调整 | [13](issues/13-bill-void-reissue-adjustment.md) | 11 |
| 14 押金退款申请、复核与执行记录 | [14](issues/14-deposit-refund-approval-execution.md) | 12 |
| 15 整合同提前终止与费用截断 | [15](issues/15-whole-contract-termination-cutoffs.md) | 09、13 |
| 16 退租结算、押金抵扣与应退抵销 | [16](issues/16-settlement-deductions-refund-offsets.md) | 14、15 |
| 17 无缝续租与连续使用 | [17](issues/17-continuous-lease-renewal.md) | 09 |
| 18 续租押金转结 | [18](issues/18-renewal-deposit-transfer.md) | 12、17 |
| 19 有关联依赖保护的收款红冲 | [19](issues/19-receipt-reversal-dependency-guards.md) | 16、18 |
| 20 招商门户与匿名留资回访 | [20](issues/20-public-portal-lead-capture.md) | 06 |
| 21 看房申请、确认与时段互斥 | [21](issues/21-viewing-appointments-slot-conflicts.md) | 20 |
| 22 手动公海、分配、领取与转交 | [22](issues/22-manual-lead-pool-assignment.md) | 06 |
| 23 企业成员与资料变更审核 | [23](issues/23-enterprise-members-profile-changes.md) | 04 |
| 24 报修、派单、完工与返工 | [24](issues/24-repairs-dispatch-completion-rework.md) | 09 |
| 25 经营、招商与资金统计 | [25](issues/25-occupancy-lead-finance-reporting.md) | 19 |
| 26 授权台账Excel导出 | [26](issues/26-authorized-excel-ledger-exports.md) | 25 |
| 27 单实例部署与备份恢复 | [27](issues/27-single-instance-deploy-backup-restore.md) | 04 |
| 28 全链路验收与交付证据 | [28](issues/28-full-flow-acceptance-delivery-evidence.md) | 21、22、23、24、26、27 |

## 覆盖与边界

每个ticket的 `User stories` 对应父spec中的连续编号，部分跨业务故事由多个ticket协作验收。共同范围以spec为准，不新增在线支付、能耗分摊、政策申报或真实电子签。

- 01通过真实登录带起最小工程，不是水平脚手架；测试主边界HTTP/API连接真实MySQL，计费快速测试和关键浏览器流程补充。
- 04建立企业资料私有文件闭环；06的跟进附件遵循相同私有契约，复用已存在的能力。它不以企业开户为业务前提，也不另建公开附件系统。
- 08完成资源接管和合同审批；后续票补实际收款、退款及履约分支，资金事项不因取消合同而消失。
- 14先完成押金退款；16接入预收租金应退、抵销和结算。所有退款执行均指核实后登记线下资金事实，不调用真实打款渠道。
- 24可先服务企业管理员，23完成受限成员后验证相应权限；17完成后回归连续使用。集成验证不制造循环阻塞。
- 27先验证当时已存在的企业与附件恢复；28在最终集成版本重跑完整合同和资金恢复。
- 28汇总回归和交付证据，不替代各ticket当场完成的权限、并发、金额及幂等测试。每项新增演示数据标为虚构，最终种子由28整理。

## 发布静态核验

- 28份独立ticket均已创建；全部open，无claimed或resolved。
- 110条用户故事全部映射，逐票编号匹配，未超出spec故事范围。
- 共258条未勾选验收条件，每票均包含完整行为、验收与上下文。
- 实际直接依赖匹配批准图，无环、无重复传递边；28的传递前置覆盖所有其余任务。
- 当前ready frontier只有01；ready-for-agent标记不解除其他任务的阻塞。
- 31份任务/索引/导航Markdown本地链接有效，父级spec的SHA256与发布前一致。

三组内容审查已完成，发现的两处边界已修正：07不以08的合同接管回归作为自身完成前置，16明确接入未签取消但已收款的退款来源；未改变批准的编号与依赖。

以上是文档静态检查；没有执行应用构建、数据库导入、业务测试、部署或恢复。

## 实现前检查

发布时仓库main尚无初始commit，应用目录尚未建立。后续使用integration branch和worktree前，需要先核实并建立包含spec、tickets及适用规则的Git baseline；提交动作需明确授权，本轮不自动提交。精确版本、DDL、完整OpenAPI与代表性UI在相应ticket内核查，不把规划写为运行事实。
