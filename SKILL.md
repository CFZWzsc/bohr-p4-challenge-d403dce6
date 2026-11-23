---
name: grant-proposal-alignment
description: Use when a case supplies grant-proposal materials (rationale, scientific question, objectives, research contents, technical route, innovation claims, preliminary work) and asks to draft or audit the proposal; builds an objective-content-indicator alignment matrix, flags unverifiable superlatives and uncited strong claims, separates completed from planned work, and writes /app/output/result.json preserving case_id and every output_contract field. Deterministic standard library only; no network.
---

# 基金申请书写作（目标—内容—路线—指标对得上）

## 用途与适用条件

当输入是基金/项目申请书的章节草稿或素材，需要检查“立得住、说得准、对得上、查得到”时使用。

适用：

- 已有立项依据、科学问题、研究目标、研究内容、技术路线、创新点、研究基础等章节文本。
- 需要把目标、内容、路线与考核指标做逐条对应，找出落单项。
- 需要清除“首次/国际领先/填补空白”等不可核验修饰。
- 需要区分“已完成”与“拟开展”，并核对论断是否有文献或数据支撑。

不适用：替代申请人确定研究方向、虚构前期成果、编造文献或政策依据、承诺经费与周期。

## 输入

- `/input/case.json`：读取 `case_id`、`output_contract`、`task`，以及 `proposal`、`sections`、`text`、`materials`、`objectives`、`milestones` 等字段。
- `task.md`：存在则读取 `/input/task.md` 或 `/app/work/task.md`，缺失不阻塞。
- 支持内联文本与文件路径；只读取存在的 UTF-8 文本（≤2MB），二进制登记为未解析。
- 输出 `/app/output`，工作 `/app/work`。

## 处理流程（逐项覆盖评分维度）

1. **章节归位（任务完成度）**：按标题关键词把段落归入 `rationale / question / objectives / contents / methods / innovation / foundation / outcomes`；无标题时按段落顺序与关键词软归类，并标 `section_inferred=true`。
2. **条目抽取（结果完整性）**：在“研究目标/研究内容/考核指标”中按编号、项目符号或句子切分条目，保留原文与字符区间。
3. **对应矩阵（对得上）**：用中英文实词与中文二元组计算重叠度，为每个目标匹配最相关内容、为每项内容匹配考核指标；低于阈值不强行配对，列入 `orphans`。
4. **修饰语审计（说得准）**：命中“首次、国际领先、填补空白、突破性、独创、国际先进、world-first、state-of-the-art”等，给出可核验的替代表述，不直接改写结论。
5. **论断—引用核对（查得到）**：识别 `[n]`、`[1-3]`、`(Author, 2020)`、`（作者，2020）`；对含“证明/表明/显著/优于”且同句无引用的句子记为 `uncited_strong_claim`。
6. **完成态区分（查得到）**：`已完成/已发表/已建立/已测得` 与 `拟/将/计划/待` 分列；研究基础章节缺完成态标记时提示 `foundation_without_completed_evidence`。
7. **结构化交付**：输出 `section_index`（原文保真）、`alignment`、`revision_checklist`（按优先级排序的可执行修改项），供 Agent 继续成稿。
8. **合同保留**：见输出契约。

## 输出契约

写入 `/app/output/result.json`：

- 顶层 `case_id` 与输入一致。
- **保留原始 `output_contract` 每个字段名**；支持字段字典/列表/`fields`/Schema `properties`。已知字段按别名填充，未映射写 `null` 并在 `contract_status.unmapped_fields` 登记，状态降为 `partial`；组合 Schema 登记 `unsupported_features`。
- 固定字段：
  - `status`：`success | partial | insufficient_input | failed`
  - `result`：`alignment`、`section_index`、`revision_checklist`、`superlative_flags`、`citation_audit`、`status_vocabulary`
  - `evidence`、`uncertainty`、`errors`、`stats`
- 矩阵条目：`objective_id`、`content_id`、`indicator_id`、`score`、`objective_text`、`content_text`、`indicator_text`、`linked`。
- 退出码：`0` 完成；`1` 有未映射合同字段；`2` 输入不可读。

## 异常处理

| 情形 | 处置 |
|---|---|
| 无可用文本 | `status=insufficient_input`，说明缺申请材料，不代写 |
| 缺“研究目标”章节 | 目标条目为空，`orphans` 记录，状态 `partial` |
| 目标多于内容 | 未配对目标逐条列入 `orphans.objectives_without_content` |
| 指标与内容口径不一致（百分比 vs 篇数） | 标 `indicator_unit_mismatch`，不擅自换算 |
| 同一目标重复表述 | 去重并记 `duplicate_objective` |
| 论断含“首次”但无检索依据 | 给替代表述，标 `needs_literature_check` |
| 研究基础引用未提供的成果 | 标 `unverifiable_preliminary_result`，不补造 |
| 材料互相冲突（周期/样本量） | 两条都保留并升级人工确认 |

## 限制与不适用

- 不检索文献数据库，不验证“首次”是否成立；只提示需要检索。
- 不计算经费预算、不承诺考核指标数值。
- 矩阵基于词面重叠，`linked=true` 只表示文本相关，不等于论证充分。
- 不替代申请人对科学问题与创新性的判断。

## 最小调用示例（示例）

以下为合成示意，非官方案例。

```bash
python /skills/weekly-skill/scripts/run_skill.py \
  --case /input/case.json \
  --out /app/output/result.json \
  --work /app/work
```

合成 `case.json`：

```json
{
  "case_id": "synthetic-grant-demo",
  "output_contract": {"status": "string", "alignment": "object", "proposal": "string", "sources": "array"},
  "proposal": "## 研究目标\n1. 建立高灵敏检测方法。\n2. 完成 200 例临床验证。\n## 研究内容\n（1）方法构建与优化。\n## 创新点\n本项目首次实现国际领先的无标记检测。"
}
```

示意输出（节选）：

```json
{"case_id":"synthetic-grant-demo","status":"success","result":{"alignment":{"matrix":[{"objective_id":"O1","content_id":"C1","linked":true}]},"superlative_flags":[{"term":"首次","section":"innovation"}]}}
```

## 自查清单

- [ ] `case_id` 与合同字段保留；未映射项已登记。
- [ ] 每个目标、内容、指标都有唯一编号，落单项已列出。
- [ ] “首次/领先/空白”类表述均有替代建议且标注需检索。
- [ ] 强论断若无引用已登记，未被写成已证实。
- [ ] 已完成与拟开展分列，未混淆。
- [ ] 未编造文献、前期成果或指标数值。


## 交付保证与评分对齐

调用顺序（固定）：

1. 先运行包内脚本（若存在）：`python scripts/run_skill.py`（默认读 `/input/case.json`，写 `/app/output/result.json`）；路径不同时用 `--case` / `--output` 指定。
2. **必须交付完整成品**：脚本只做章节归位与对齐审计；调用 Agent 必须输出完整 `alignment` 矩阵、`revision_checklist`（按优先级、可执行）与 `section_index`；任务要求成稿时须把修订后的申请书正文写入 `result.proposal`，目标—内容—指标逐条对应；不得只交诊断、`pending` 或空章节。
3. 最后自检：`case_id` 与输入一致；`output_contract` 每个字段都存在；`status` 取本合同允许值 `success | partial | insufficient_input | failed`；正文无 `pending`/`TODO`/空占位；数字、引用、来源可逐条回溯。

运行保证：

- 输入缺失、JSON 损坏、字段异常、输出目录不存在时，仍须写出结构化 `result.json` 并正常结束，不得崩溃或非零退出。
- 不联网、不伪造数据、不声称平台评分或官方案例结果。
- 交付物必须是可直接编辑与复核的成品，而不是流程说明。

评分对齐（本题五个维度）：

- 实际效果（40）：完整产出任务要求的最终成品，覆盖全部输入条目，可直接进入修改流程。
- 使用增益（25）：体现相对基础 Agent 的可验证改进（结构化、术语/数字/引用核对、可编辑性）。
- 创新增益（15）：保留方法学增量（分块、对齐、二次校验、版本比对等），并用结果证明其价值。
- 可靠体验（10）：异常输入有明确降级与说明，不同长度/文体/重复运行下稳定产出。
- 生态贡献（10）：用途、调用、输入输出、限制、示例、维护信息齐全，资源可被他人复用。

## 最终交付强制要求

1. **调用 Agent 必须交付成品，不是流程说明**：脚本产出只是索引、核对或规格骨架；调用 Agent 必须在同一 `result.json` 中补齐本题要求的最终产物（正文/译文/审稿意见/综述/分析结论/图表源文件与导出件/幻灯片页内容与讲稿/实验方案等）。交付字段为空、只有标题骨架，或含 `pending`、`TODO` 之类占位，一律视为未完成。
2. **必须覆盖全部输入条目**：输入中的每个章节、问题、文献、图、页面、指标都要有对应条目，不得只处理示例或首条；确实缺失的条目写入 `uncertainty` / `limitations` 并说明原因，不得静默跳过。
3. **必须自检后再交付**：`case_id` 与输入一致；`output_contract` 每个字段都存在；状态值取本题契约规定的枚举；数字、单位、引用、来源可逐条回溯；结果内部无自相矛盾。
4. **参数等价**：`python scripts/run_skill.py --case <输入> --out <输出>` 与 `--output <输出>` 等价；默认 `/input/case.json` → `/app/output/result.json`。
5. **平台用 3 个固定案例分别运行**：逻辑要由输入驱动，不针对某一份示例硬编码；字段命名差异走别名匹配与显式降级，并在 `uncertainty` 记录假设。
