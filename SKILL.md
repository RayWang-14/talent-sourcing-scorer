---
name: talent-sourcing-scorer
description: Assess whether a person is worth prioritizing for early-stage VC sourcing based on public information, with identity resolution first, structured evidence collection, concise Chinese output, and a practical scorecard. Use when Codex needs to evaluate a founder, researcher, engineer, operator, or emerging talent from open sources, especially when names may be ambiguous across Chinese, English, or pinyin variants.
---

# Talent Sourcing Scorer

帮助早期 VC 基于公开信息，快速判断一个人是否值得优先 `sourcing`。默认输出语言为中文；只有必要的专有名词、机构名、论文名、项目名、职位名、技术术语保留英文，优先采用“中文解释 + 英文原词)”的写法。

## 什么时候调用

在以下情况调用本 skill：

- 需要基于公开信息，快速初筛某个人是否值得优先接触
- 需要先做身份校验（identity resolution），避免把同名同姓或中英文别名不同的人混在一起
- 需要把 GitHub、论文、学校主页、公司 bio、社交媒体等公开信息汇总成结构化结论
- 需要面向中国早期 VC 产出简洁、结构化、可快速阅读的中文报告
- 需要输出 `overall score`、`confidence score`、`false-match risk`、`sourcing priority`

## 什么时候不要调用

在以下情况不要调用，或只把它当作辅助信息：

- 需要正式背调、合规审查、雇佣决策、投资决策结论
- 需要依赖私有数据、付费数据库、未公开材料、内部 CRM 信息
- 需要判断人格、诚信、管理风格等无法由公开信息可靠推出的内容
- 需要给出“绝对排名”或精确财务、薪酬、股权信息
- 只有极弱线索且用户不允许保守输出

## 输入

最少输入：

- `target_name`：目标姓名，可为中文名、英文名、拼音名，或混合写法

强烈建议补充：

- `current_org`：当前公司 / 学校 / 实验室 / 项目线索
- `known_anchor`：任一锚点，例如学校、城市、GitHub handle、论文题目、项目名
- `goal`：为什么要评估这个人，例如“是否值得优先约聊”

可选输入：

- `alt_names`：别名、英文名、曾用名
- `location`：城市 / 国家
- `domain_hint`：研究方向 / 技术方向 / 职位方向
- `known_urls`：已知个人主页、LinkedIn、GitHub、Google Scholar 等链接
- `time_hint`：时间线线索，例如“2024 年在 Moonshot AI”

如果只有名字且高度歧义，先停在“候选身份列表 + 初步判断”，不要强行进入评分。

## 默认输出

默认使用 [report_template.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/report_template.md) 的结构，输出以下字段：

1. 评分卡 / scorecard
2. 一句话结论 / executive summary
3. 身份校验总结
4. 结构化证据
5. 为什么值得优先 sourcing / 为什么暂时不用
6. 首次交流最值得验证的未知点
7. 重要来源

最终交付物必须是一个 `.md` 文件，不要只在对话里输出。

默认文件名：

- `<中文名>-<English-Name>-<overall_score>.md`
- 例如：`陈广宇-Nathan-Chen-87.md`

命名规则：

- 中文名保持原样
- 英文名中的空格改成 `-`
- 分数使用整数
- 如果身份未稳定锁定、暂不该出总分，改用 `<中文名>-<English-Name>-identity-unresolved.md`

必须显式输出：

- `match confidence`
- `false-match risk`
- `evidence density`
- `overall score`
- `confidence score`
- `sourcing priority`

字段名可保留英文；解释、判断、风险提示、结论必须主要使用中文。

## 核心工作流

严格按以下顺序执行。

### 第一步：身份校验

把身份校验放在最前面，而且视为最重要环节。

执行要求：

- 先验证身份，再合并资料，再评分
- 建立候选身份列表，按“最可能是目标人物”排序
- 至少有两个以上一致锚点，才允许把不同来源的信息合并到同一个人身上
- 可用锚点包括：学校、公司、实验室、研究方向、城市、个人主页、GitHub handle、Google Scholar 页面、论文题目、项目名、头像、时间线
- 如果发现同名冲突、时间线冲突、机构冲突、研究方向明显冲突，默认不要强行合并
- 缺乏足够证据时，允许保守，不允许脑补

输出要求：

- 说明你锁定的是哪个候选身份
- 明确列出关键锚点
- 给出 `match confidence`、`false-match risk`、`evidence density`
- 如果仍有多个候选身份，列出每个候选身份及其支持证据，并指出哪个更可能是目标人物

### 第二步：公开信息搜集

优先搜集高可信、一手、可交叉验证的信息。重要结论尽量附来源。

至少覆盖以下公开来源：

- GitHub
- LinkedIn
- 个人主页
- 学校主页 / 实验室主页
- Google Scholar
- DBLP / Semantic Scholar / arXiv
- X / Twitter
- 博客 / 采访 / 播客 / 公司官网 bio
- 公开奖项、奖学金、竞赛记录、专利、媒体报道

可按需要补充高价值来源：

- OpenReview
- Hugging Face
- ORCID
- Crunchbase 或公司官网团队页
- YouTube / conference talk 页面

来源优先级：

1. 个人主页、学校 / 实验室主页、公司官网 bio、GitHub、Google Scholar、DBLP
2. 论文数据库、专利、公开奖项、官方采访、会议主页
3. X / Twitter、博客、播客、媒体报道、二次转载

### 第三步：结构化总结

把事实和判断分开写。

必须区分：

- `public visibility` 和真实能力
- 团队成就和个人贡献
- 学术强、工程强、商业化潜力强，这三类信号
- 机构光环和个人能力

建议归纳的证据块：

- 当前身份与时间线
- 研究与论文
- 工程与产品
- 奖项、竞赛、奖学金、稀缺背景
- 对外影响力与叙事能力
- 近几年 trajectory

### 第四步：打分与风险提示

评分目标不是做学术排名，而是判断“是否值得优先花时间接触”。

评分时：

- 不要把机构光环直接等同于个人能力
- 不要把 GitHub star、论文引用、社交媒体热度中的任何一个单指标当作结论
- 不要把团队成果默认归功为个人核心贡献
- 对证据不足的维度允许低置信、允许留白

读取 [references/scoring_framework.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/references/scoring_framework.md) 后再输出分数和优先级。

## 评分框架

评分维度如下：

- 研究影响力
- 工程与产品能力
- 稀缺与 outlier signal
- 影响力与叙事能力
- trajectory
- sourcing fit

细则、权重、优先级映射写在 [references/scoring_framework.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/references/scoring_framework.md)。

## 常见误判和边界情况

- 中文名、英文名、拼音名混用，导致把不同人并到一起
- 同名论文作者、同名 GitHub 账号、同名 LinkedIn 资料误合并
- 学校主页或公司 bio 过期，时间线未更新
- GitHub star 很高，但主要是团队项目或短期热点，不代表个人长期工程能力
- 论文引用很高，但作者位置弱、个人贡献不清，或领域引用基准不同
- 社交媒体热度很高，但更多是内容传播而非技术、研究、产品能力
- 团队荣誉、奖项、媒体报道被误写成个人主导成果
- 实习、访问学者、顾问、志愿者身份被误判为正式核心任职
- 二手媒体和自我介绍相互引用，形成“看起来很多来源，实际上同源”的假象

## 输出原则

- 默认输出语言为中文
- 结论先行，但不要过度断言
- 重要结论尽量附来源
- 当证据不足时，明确写“暂无法确认”
- 当身份未稳定锁定时，不输出确定性总分，只输出候选身份和后续验证建议
- 评分卡放在最前面，一句话结论放在第二部分
- 最终报告保存成一个独立 `.md` 文件

## 资源

- 使用 [report_template.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/report_template.md) 组织最终报告
- 使用 [references/scoring_framework.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/references/scoring_framework.md) 进行打分
- 参考 [example_output.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/example_output.md) 的版式和语气
- 使用 [test_cases/case_01_name_collision.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/test_cases/case_01_name_collision.md)、[test_cases/case_02_research_engineering.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/test_cases/case_02_research_engineering.md)、[test_cases/case_03_operator_builder.md](/c:/Users/wr981/Desktop/talent-sourcing-scorer/test_cases/case_03_operator_builder.md) 做快速试跑
