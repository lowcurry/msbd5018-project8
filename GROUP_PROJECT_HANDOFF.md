# MSBD 5018 Group 16 · Project 8 进度交接

**更新于 2026 年 11 月 8 日（香港时间）**

课题为 *Evaluating Jev for RAG Citation Verification*。主实验评估 Jev 对单条论断和单段证据的关系判断，四模型扩展评估用 Jev 重排检索证据后能否改善问答结果。

## 1. 目前进度

| 工作 | 状态 | 入口 |
|---|---|---|
| VitaminC 数据与实验冻结 | 已完成。开发 200 条、校准 2,000 条、选择 2,000 条、主测试 5,000 条，四组按文章隔离。模型、阈值、输入与评分方案于 11 月 6 日 20:06 冻结。 | [冻结记录](results/dev/protocol_freeze.json)、[数据来源](data_source/README.md)、[实验状态](STATUS.md) |
| 模型比较与误差分析 | 已完成原标签上的自动评估、文章配对 bootstrap、预算消融和部分错误案例分析。开发选择集上 FLAN-T5-base 优于 small，因此选 base 作为回退模型。 | [主结果](results/MAIN_RESULTS.md)、[错误案例](results/test_case_analysis.md) |
| 真实链路计时 | 已完成。同一组 100 条样本，每种方案重复三次，并发数为 1。 | [计时汇总](benchmark/real_chain_comparison.csv)、[资源账](results/resources/resource_summary.csv) |
| 主测试标签审核 | 600 条盲审包已准备。待 A、B 独立标注，C 盲裁决，再用冻结预测补算。 | [盲审流程](handoff/README.md)、[标注规则](ANNOTATION.md) |
| 语言对照诊断 | 200 组、400 条预测已完成。待独立审核标签和编辑有效性，再统计有效组表现。 | [标注规则](ANNOTATION.md) |
| 四模型 RAG 扩展 | 数据转换、配对运行、Jev 重排和评分代码已写好。用合成数据验证了计划检查和评分入口。真实问答批次尚无结果，四家端点与环境变量名待配置。 | [扩展实验说明](rag_experiment/README.md)、[配置模板](rag_experiment/config.json) |
| 报告与汇报 | ACL PDF、LaTeX 源码和 PPT 均已有自动评估版。人工结果出来后更新。 | [报告草稿](report/Project8_ACL_Automated_Results.pdf)、[报告源码](report/main.tex)、[演示稿](presentation/Project8_Automated_Results.pptx) |

## 2. 可用于 Results 阶段的分数

5,000 条冻结 VitaminC-real 测试，按数据集原始标签评分。标签为支持（S, Support）、矛盾（C, Contradiction）与证据不足（I, Insufficient evidence）。

| 方法 | Macro-F1 | Accuracy | FLAN 调用次数 |
|---|---:|---:|---:|
| Jev 1.13 Choice | 0.8126 | 0.8386 | 0 |
| DeBERTa NLI | 0.5052 | 0.5072 | 0 |
| 全量 FLAN-T5-base | 0.2575 | 0.3058 | 5,000 |
| 冻结 Jev → FLAN | 0.8124 | 0.8386 | 2 |
| 冻结 DeBERTa → FLAN | 0.5052 | 0.5072 | 0 |

- 冻结 Jev 路由相对全量 FLAN 的 Macro-F1 差为 **+0.5548**，按文章配对 bootstrap 得到双侧 95% 区间 **[+0.5380, +0.5708]**。该结论适用于所选 FLAN-T5-base，其支持类召回率约 **2.55%**。[测试指标](results/test/metrics_main.csv) · [区间数据](results/test/bootstrap_primary.json)
- Jev 的两次升级都由请求失败触发，回退未纠正这两条。冻结路由的 Macro-F1 略低于独立 Jev。[升级分解](results/test/escalation_decomposition.csv)
- Jev 路由端到端延迟为 **692／1,113 ms**（p50／p95），全量本地 FLAN 为 **104／122 ms**。本次路由减少了 FLAN 调用，实测端到端耗时更长。[计时结果](benchmark/real_chain_comparison.csv)

## 3. 组员接手顺序

| 角色（待组内分配） | 接手动作 | 完成后交回 |
|---|---|---|
| 数据协调者 | 组织 40 条训练样本统一标注口径，随后向 A、B 单独发包。盲审期间单独保管逐条原标签、预测、编号映射及组对照信息。 | 分发记录、冻结材料、归档后的标注和裁决文件 |
| 标注者 A、B | 各自审核 600 条主测试样本与 400 条诊断成员。先交独立标签，再检查 200 个诊断组是否隔离数字、时间、实体或否定变化。 | 本人 CSV，保留原 ID 与文本，填写标签、证据理由和是否此前见过 |
| 裁决者 C | 接收协调者导出的匿名分歧与歧义项，按同一规则裁决并记录未决或已见样本。 | 裁决 CSV 及理由 |
| 评估与写作负责人 | 协调者用 `human.py merge` 合并审核，再运行 `evaluate.py test` 的人工标签分支。根据一致性、覆盖率、原标签与裁决结果的差异更新报告和 PPT。 | 人工审核汇总、更新后的 ACL PDF 和 PPT |
| RAG 扩展负责人 | 固定 HotpotQA distractor 开发集抽样清单，配置并验证 GLM、Kimi、DeepSeek、Qwen 和 Jev，先小批试跑确认服务端模型名，再跑正式配对实验。 | 固定清单、配置版本、配对分数与区间、失败率、耗时及证据召回 |

扩展实验默认取 200 题。基线从每题的 10 段候选按 BM25 取前 5 段，实验组让 Jev 对同一批候选重排后取前 5 段。四个回答模型各跑两组，预计产生 **1,600 个回答试验**和 **2,000 个共享 Jev 评分请求**。主要指标是四模型等权平均的答案精确匹配率（Exact Match）。扩展结果单列。ALCE 外部引用验证暂缓，已有筛选候选仍需核对出处与 claim 边界，并独立标注 S/C/I。

## 4. 复现与文件交接

- 组员仓库包含源代码、配置模板、汇总分数、数据出处和报告草稿。[复现说明](README.md)
- 协调者保管冻结数据 `data/`、逐条预测 `predictions/` 和审核 CSV。盲审组员按分配仅接收对应审核包与 rubric。人工审核结束后，协调者可用现有缓存补算，无需重新调用模型。
- 冻结主测试复算要求 **Python 3.12.13、NumPy 2.5.4**，以及与冻结记录匹配的输入、缓存和代码。交接时核对了评估脚本及运行记录的哈希，并逐项对照现有汇总指标。当前运行环境的版本不符，冻结校验拒绝了完整复算。
- API 密钥通过本机环境变量配置。四模型实验可以将 `rag_experiment/config.json` 复制为被 Git 忽略的 `rag_experiment/config.local.json`，运行时用 `--config` 指定该文件。配置中填写端点和环境变量名。

## 5. 课程提交节点

| 节点（香港时间） | 提交物与来源 |
|---|---|
| 11 月 17 日 23:59 | Canvas 提交主要评价分数。使用冻结主测试指标，注明目前按原标签评分，并核对 Canvas 的示例格式。 |
| 12 月 9 日 23:59 | 官方 ACL 模板报告，最多 8 页，以及可复现代码和数据或可访问链接。协调者负责提供提交用完整材料。 |
| 时间待课程公告 | 演示稿按指南约 5 分钟准备，以 Canvas 最终要求为准。 |
