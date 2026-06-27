# Grant-Master Demo

这个 demo 用于演示 Grant-Master 的自动模式。

## 目标

- 使用自动模式运行 Grant-Master。
- 最小调研轮数设置为 1 轮。
- 生成一份约 1 万字的中文项目申请书。
- 本目录用于展示一次完整 workflow 的运行方式和关键产物。
- 当前 `workflow/` 中的所有内容均由一次自动模式对话直接生成，全程大约 1 小时，没有追加任何额外指令。
- 出于版权和体积考虑，论文 PDF 未随仓库分发；本 demo 展示的是完整流程记录和最终 docx，不是可重新精读 PDF 的完整归档。

## 运行方式

在本目录下运行：

```text
/grant-master:auto --auto 最小调研轮数 1 轮，目标生成约 10000 字的中文项目申请书
```

运行前可以检查或修改 `topic.md`。Grant-Master 会在当前目录下生成或更新 `workflow/`、`papers/` 等过程文件和结果文件。

## 关键产物怎么看

如果只想快速了解 demo 的结果，建议优先看这几类文件：

- `workflow/05_synthesis/current_view.md`：领域综合理解。这里汇总了前期文献调研和论文精读后的核心问题、已有工作、证据链、领域 gap 和后续方案判断，是最适合学习新方向的材料。
- `workflow/07_outline/outline_blueprint.yaml`：申请书大纲蓝图。这里能看到申请书的章节结构、论证链条和整体组织方式；更细的正文拆解可继续看 `workflow/07_outline/writing_units.yaml`。
- `workflow/11_output/proposal.docx`：最终 Word 输出。这里是自动流程跑完后的申请书初稿，可用于整体预览内容、格式和章节完整性。

## 使用建议

如果你的目标是学习一个新方向，而不是立刻生成完整申请书，完全可以只跑到 `06-helm` 就停下来。此时 `05_synthesis/current_view.md` 和 `06_helm/helm_report.md` 已经包含了比较完整的领域理解、问题判断和技术路线收敛过程。

如果你的目标是写申请书，请把自动生成结果当作初稿。后续可以围绕 `07_outline/outline_blueprint.yaml`、`07_outline/writing_units.yaml` 多轮修改大纲、章节体量和论证顺序，也可以针对 `workflow/08_section_write/units/` 中的正文单元逐段修改、重写或补充材料，再重新执行组装、审阅和输出阶段。

## 注意

本 demo 不会自动执行。请由使用者在自己的 Codex 或 Claude Code 环境中手动运行上面的命令。

自动生成的申请书不能直接提交。正式使用前，请务必进行事实核验、引用核验、政策合规检查、格式检查和人工审阅。
