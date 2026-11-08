# Issue tracker: Local Markdown

任务和 spec 保存在本仓库 `.scratch/` 中。

## 文件约定

- 每个 feature 使用 `.scratch/<feature-slug>/`。
- spec 使用 `.scratch/<feature-slug>/spec.md`。
- 每个 ticket 单独使用 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，从 `01` 编号。
- ticket 顶部使用 `Status:` 记录状态；未启用 triage label 映射。
- 评论和讨论追加到文件底部的 `## Comments`。

## Skills 操作

“publish to the issue tracker”表示在对应 feature 目录创建 Markdown 文件。
“fetch the relevant ticket”表示读取用户指定的 ticket 路径或编号。

## Wayfinding

- map 使用 `.scratch/<effort>/map.md`，记录 Notes、Decisions-so-far 和 Fog。
- 子 ticket 使用 `.scratch/<effort>/issues/NN-<slug>.md`。
- `Type:` 使用 `research`、`prototype`、`grilling` 或 `task`。
- `Status:` 使用 `open`、`claimed` 或 `resolved`。
- `Blocked by: NN, NN` 记录依赖；依赖全部 `resolved` 后才能领取。
- 从 `open`、无阻塞的 ticket 中按编号领取，开始前保存为 `claimed`。
- 完成后在 `## Answer` 追加结果，设为 `resolved`，并向 map 的 Decisions-so-far 追加结论和文件链接。
