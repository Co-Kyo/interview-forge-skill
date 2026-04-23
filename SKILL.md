---
name: interview-forge
description: 面试模拟答题系统。当用户说"开始面试"、"模拟面试"、"面试练习"、"面试答题"、"做面试题"、"面试模拟"时触发阶段1(出题+启动);当用户说"答题完成"、"答完了"、"面试完成"、"生成报告"时触发阶段2(归因+报告);当用户说"建档案"、"创建档案"、"知识追踪"、"学习档案"、"我要建档"时触发档案初始化;当用户说"关闭面试"、"关掉面试"、"关端口"、"停掉面试"时关闭 Vite 服务。
---

# InterviewForge 面试模拟 Skill

**Copyright (c) 2026 Co-Kyo - Licensed under CC BY-NC 4.0**

本文件及所有包含的出题流程规范、算法设计、知识模型为原创工作,

前端仓库地址:https://github.com/Co-Kyo/interview-forge-public.git

---

## 概述

InterviewForge 是一个本地面试模拟答题系统。本 Skill 分两阶段工作:

- **阶段1(出题+启动)**:从对话上下文提取面试需求 → 生成题库 → 启动答题系统
- **阶段2(归因+报告)**:读取答题结果 → 归因分析 → 生成雷达报告 HTML

两阶段之间由用户手动衔接--答完题回到 WorkBuddy 说"答题完成"即可触发阶段2。

## 触发条件

### 阶段1 触发词
- "开始面试" / "模拟面试" / "面试练习"
- "面试答题" / "做面试题" / "面试模拟"

### 阶段2 触发词
- "答题完成" / "答完了" / "面试完成"
- "生成报告" / "出报告"

### 档案初始化触发词
- "建档案" / "创建档案" / "建立学习档案"
- "知识追踪" / "学习档案" / "我要建档"

### 关闭面试触发词
- "关闭面试" / "关掉面试" / "关端口"
- "停掉面试" / "停止面试" / "结束面试"(非归因语义,而是主动关进程)

> **语义区分**:"答题完成"/"答完了"是触发归因(阶段2),用户还有答题结果要处理;"关闭面试"/"关端口"是纯运维操作,只杀 Vite 进程,不触发归因。

### 自动判断
如果用户在 InterviewForge 启动后回来,且档案目录的 `sessions/{date}/` 下存在新的 `result-*.json`,自动进入阶段2。

---

## 前置步骤0:环境检查与项目部署

**每次 Skill 启动时,必须先执行环境检查。**

读取文件:`{SKILL_DIR}/setup/env-check.md`

该文件包含:
- 平台检测(OS / Git / Node.js ≥ 20)
- 平台变量设置(Shell 包裹方式 / nvm 激活 / 路径分隔符 / 杀端口命令)
- 前端项目部署检查与自动 clone
- Node.js 版本安装

**按其中流程依次执行,任何一项阻断则不继续。**

---

## 前置步骤1:档案路径检查

**每次 Skill 启动时,必须先检查是否已配置知识追踪档案路径。**

### 检查逻辑

1. **检索持久化记忆**:通过 `conversation_search` 查询 `interview-forge-archive-path`
2. **已配置档案路径** → 读取路径,设为 `{ARCHIVE_PATH}`,跳过本步骤,直接进入阶段1
3. **未配置档案路径** → 执行档案初始化流程

### 档案初始化流程

读取文件:`{SKILL_DIR}/references/phase-minus1-profile-setup.md`

按该文件中的完整流程执行:
1. 询问用户档案路径(**必须指定,不设默认值**)
2. 验证路径合法性(可写、不存在则询问创建)
3. 创建目录结构 + git init + 初始文件
4. 通过 `update_memory` 持久化档案路径
5. 告知用户初始化完成

> **注意**:档案路径一旦设定,贯穿所有后续 session。用户说"换一个档案路径"时才重新初始化。

---

## 阶段1:出题 + 启动

### Step 0:提炼测试目标

**出题前必须先从用户需求中提炼出最小核心测试目标,目标决定出题策略。**

读取文件:`{SKILL_DIR}/references/phase0-direction-assessment.md`

该文件包含:
- 三层推断模型(KAOS目标精炼 + GROW收束链 + 洋葱模型)
- A 逻辑:从用户话语中提取三类信号(话题/目的/差距)→ 自动推断测试目标
- B 逻辑:A 无法收束时,主动问用户"你要什么结果"来补充信息
- 测试目标 → 出题策略的映射表
- 复合目标处理规则

**判定流程**:
1. 读取 direction-assessment.md 中的推断模型和信号映射表
2. 扫描当前对话上下文,提取三类信号(话题/目的/差距)
3. A 逻辑判定:
   - 三信号齐全 → 直接提炼目标 → 进入 Step 1
   - 有话题+目的,无差距 → 提炼主目标 + 关联辅助目标 → 进入 Step 1
   - 只有话题 → 进入 B 逻辑
4. B 逻辑:用 ask_followup_question 弹出选项卡确认测试目的 → 补充信息带回 A 逻辑 → 提炼目标 → 进入 Step 1

> **注意**:测试目标一旦确定,贯穿本次会话全程(出题+归因),除非用户明确要求切换。

### Step 1:拉取出题格式规范和逆向拆解指南

**必须先读取本 Skill 的出题格式规范和逆向拆解指南,再生成题库。**

读取文件:
- `{SKILL_DIR}/Tools/interview-forge-public/schemas/quiz.schema.json` - 题库 JSON 结构定义（权威源，生成结果必须通过此 Schema 校验）
- `{SKILL_DIR}/references/phase1-quiz-format.md` - 出题语义规则（Schema 无法表达的补充规则）
- `{SKILL_DIR}/references/phase1-goal-decomposition.md` - 目标逆向拆解指南
- `{SKILL_DIR}/references/phase1-knowledge-lifecycle.md` - 知识生命周期模型(**必读**)

quiz.schema.json 定义题库的结构合同（字段、类型、必填、枚举、条件约束），前端项目以此作为数据消费的唯一真相源。
phase1-quiz-format.md 补充 Schema 无法表达的语义规则：质量要求、编排规则、答案分布、lifecycle 审计等。

**生成题库后，必须用 quiz.schema.json 对输出做结构性校验**，确保 JSON 合法再写入文件。

### Step 2:基于上下文和测试目标生成题库

**必须先读取目标逆向拆解指南,按拆解逻辑出题,而非随意生成。**

1. **逆向拆解知识组件**:按 goal-decomposition.md 第一步,从测试目标逆向拆解出 8~15 个知识组件
   - 面试叙事→按因果链拆解,诊断定位→按知识域拆解,概念辨析→按差异维度拆解,案例推导→按推理步骤拆解,知识校准→按掌握层级拆解
2. **映射证据需求**:按 goal-decomposition.md 第二步,为每个组件确定证据类型(识别/辨析/解释/应用/推导)
3. **设计题目**:按 goal-decomposition.md 第三步,从证据需求出发设计具体题目
   - 一题一组件,证据类型决定题型,干扰项从"常见误解"中取
   - 主目标题 60~70% + 辅助目标题 30~40%
   - 每道题标注所属测试目标维度,归因时按维度分组
   - 不同测试目标的选项设计模板见 goal-decomposition.md 第三步
4. **按照 phase1-quiz-format.md 的格式约束组装题库 JSON**
5. **质量检查(生成后必须执行,不可跳过)**:
   - **知识覆盖检查**:每个知识组件是否至少有 1 道题覆盖?是否有组件遗漏?
   - **证据对齐检查**:每道题的证据类型是否与组件需求匹配?干扰项是否来自"常见误解"?
   - **lifecycle 标注检查**(参照 phase1-knowledge-lifecycle.md):
     - 每道题是否标注了 `lifecycle` 字段?
     - declining 阶段的知识点,stem 是否包含版本上下文?
     - deprecated 阶段的知识点,是否被错误地用作主考题?
     - 同一 category 下 declining 题目占比是否超过 30%?
     - 同一知识点在不同题中的 lifecycle 标注是否一致?
   - **答案分布检查**:统计所有选择题的正确答案分布,任意选项占比不得超过 40%
   - **选项有效性检查**:4 个选项都有竞争性,有且仅有一个正确答案
   - **代码块渲染检查**:stem 和 options 中的代码片段是否用了 `` ``` `` 围栏代码块语法?
   - **策略对齐检查**:题型比例/理由必填/交叉检验是否与测试目标对齐
   - meta.totalQuestions 与 questions 数组长度一致,ID 从 q01 连续编号

### Step 3:写入题库文件并启动 InterviewForge

1. **生成 sessionId**:格式 `if-{YYYYMMDD-HHmmss}`(用当前时间戳)

2. **确保会话目录存在**:读取 `{SKILL_DIR}/setup/frontend-launch.md`,按其中「创建会话目录」流程执行

3. **写入题库文件**:将生成的 JSON 写入 `{ARCHIVE_PATH}/sessions/{date}/quiz-{sessionId}.json`

4. **启动 InterviewForge**:读取 `{SKILL_DIR}/setup/frontend-launch.md`,按其中「启动 InterviewForge」流程执行

5. **启动前清理端口**:读取 `{SKILL_DIR}/setup/port-clean.md`,按其中「启动前清理端口」流程执行

### Step 4:告知用户

```
✅ 面试已启动!浏览器将自动打开答题页面。

答完所有题目后,回到这里告诉我"答题完成",我会为你生成归因雷达报告。
如果中途需要关闭,随时说"关闭面试"即可停掉服务。
```

---

## 阶段2:归因 + 报告

### Step 1:定位答题结果

1. **扫描结果目录**:查找 `{ARCHIVE_PATH}/sessions/*/result-*.json`(递归扫描所有日期子目录)
2. **取最新的 result 文件**(按文件修改时间排序)
3. **验证 sessionId**:如果有多个 result 文件,优先匹配阶段1 启动时的 sessionId
4. **如果找不到 result 文件**:
   - 询问用户是否已完成答题
   - 如果未完成,提示继续答题后回来

### Step 2:归因分析 + 生成雷达报告

**不使用脚本,直接用大模型能力完成归因和报告生成。**

#### Step 2a:读取归因参考文件

**必须先读取归因参考文件,再执行归因分析。**

读取文件:
- **`{SKILL_DIR}/references/phase2-attribution-prompt.md`**(必读)- 执行编排层:prompt 结构 + 数据注入 + 输出格式
- **`{SKILL_DIR}/references/phase2-attribution-guide.md`**(必读)- 权威定义层:所有归因标签、规则、示例的唯一权威来源

> 归因标签定义以 `attribution-guide.md` 为准,`attribution-prompt.md` 只引用不重复。

#### Step 2b:执行归因 + 写入结果

1. **读取数据文件**(从档案目录读取):
   - 题库文件:`{ARCHIVE_PATH}/sessions/{date}/quiz-{sessionId}.json`
   - 结果文件:`{ARCHIVE_PATH}/sessions/{date}/result-{sessionId}.json`

2. **组装 Prompt**:
   - 读取 `attribution-prompt.md` 内容
   - 将 `{QUIZ_JSON}` 占位符替换为题库文件内容
   - 将 `{RESULT_JSON}` 占位符替换为结果文件内容

3. **提交大模型执行**:将组装后的完整 prompt 提交给大模型

4. **解析输出**:大模型只输出结构化 JSON(以 ```json 代码块包裹),**不再输出 HTML**

5. **写入归因 JSON**:JSON → `{ARCHIVE_PATH}/sessions/{date}/attribution-{sessionId}.json`

6. **调用渲染脚本生成 HTML 报告**:

**macOS / Linux**:
   ```bash
   node "{SKILL_DIR}/Tools/scripts/render-report.cjs" --json "{ARCHIVE_PATH}/sessions/{date}/attribution-{sessionId}.json" --output "{ARCHIVE_PATH}/sessions/{date}/report-{sessionId}.html"
   ```

**Windows**(通过 Git Bash):
   ```bash
   & 'C:\Program Files\Git\bin\bash.exe' -c "node '{SKILL_DIR}/Tools/scripts/render-report.cjs' --json '{ARCHIVE_PATH}/sessions/{date}/attribution-{sessionId}.json' --output '{ARCHIVE_PATH}/sessions/{date}/report-{sessionId}.html'"
   ```
   > **关于 .cjs 扩展名**:前端项目的 `package.json` 声明了 `"type": "module"`,而渲染脚本使用 CommonJS 语法(`require`),因此脚本文件使用 `.cjs` 扩展名,确保 Node.js 按 CJS 模式解析,无需运行时复制。

> **Token 优化**:改由脚本渲染 HTML 后,归因阶段 LLM 不再生成 ~8K tokens 的 HTML 骨架/CSS/JS,节省约 70% 的归因 token 消耗。

### Step 3:Git 归档

**每次答题完成后,自动将本次会话数据提交到 Git。**

**macOS / Linux**:
```bash
cd "{ARCHIVE_PATH}" && git add . && git commit -m "session: {sessionTitle} - {date}"
```

**Windows**(通过 Git Bash):
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "cd '{ARCHIVE_PATH}' && git add . && git commit -m 'session: {sessionTitle} - {date}'"
```

> 如果 Git 未安装或未初始化,跳过此步骤并提示用户。

### Step 4:告知用户 + 基于 diagnosisType 提议后续行动

输出报告路径,打开报告,然后**根据归因结果中的 diagnosisType 差异化提议后续行动**:

#### diagnosisType = knowledge-gap

```
✅ 归因报告已生成:
   📊 HTML 报告:{report 文件路径}
   📋 结构化数据:{attribution JSON 文件路径}

📋 报告中识别到知识缺口({列出 knowledge-gap 的维度}):
   - {recommendedAction = supplement-article}:已有官方文档/权威资料可覆盖,我可以直接补充文章
   - {recommendedAction = deep-research-needed}:超出已有知识库范围,需要深度研究收集

   回复"补充文章"让我补充已有覆盖的缺口,回复"深度研究"启动针对性研究,回复其他内容则跳过。
```

#### diagnosisType = precision-gap

```
✅ 归因报告已生成:
   📊 HTML 报告:{report 文件路径}
   📋 结构化数据:{attribution JSON 文件路径}

📋 报告中识别到认知精度问题({列出 precision-gap 的维度}):
   文章已有覆盖但你理解不够精确,建议再测一轮同维度题确认深度。

   回复"再测一轮"启动验证测试,回复其他内容则跳过。
```

#### diagnosisType = process-defect

```
✅ 归因报告已生成:
   📊 HTML 报告:{report 文件路径}
   📋 结构化数据:{attribution JSON 文件路径}

📋 报告中识别到出题流程问题({列出 process-defect 的维度}):
   部分题目的版本上下文缺失可能影响了你的答题表现,我会修复出题流程后重新出题。

   回复"修流程重测"启动修正后的测试,回复其他内容则跳过。
```

#### 混合型(多种 diagnosisType 并存)

按优先级排序提议:`process-defect`(先修流程) > `knowledge-gap`(再补知识) > `precision-gap`(最后再测),一次性列出所有建议行动。

### Step 5(可选):深度研究推进

**仅在用户确认后执行。** 基于归因报告中的 P0 级薄弱点,执行针对性深度研究。

#### 执行流程

1. **从归因结果提取 P0 主题**:从 Step 2 的归因分析中提取标记为 P0 的知识盲区,作为研究方向

2. **收集一手资料**(按优先级):
   - Vue.js 官方文档(vuejs.org)
   - GitHub 源码(vuejs/core 仓库)
   - MDN / dev.to / Medium 等权威社区
   - 跳过域名:zhihu.com、csdn.net、jishuzhuan.net、bytezonex.com、cnblogs.com
   - 降级策略:web_fetch 失败立即换源,同一 URL 不重试,连续 2 次失败换方向

3. **产出 Markdown 文档**:
   - 写入 `{ARCHIVE_PATH}/deep-research-{sessionId}.md`
   - 每个 P0 主题一个章节,包含:盲区诊断、源码级解析、时序/结构图、逻辑链条、行动计划
   - 末尾附参考资料(标注平台·作者·原创)

4. **告知用户**:
```
✅ 深度研究已完成:{deep-research 文件路径}
```

---

## 运维操作:关闭面试

**触发词**:"关闭面试" / "关掉面试" / "关端口" / "停掉面试" / "停止面试"

此操作仅关闭 Vite 开发服务器进程,**不触发归因**。

读取文件:`{SKILL_DIR}/setup/port-clean.md`,按其中「关闭面试服务」流程执行。

---

## 注意事项

### 流程性约束

1. **出题前必须先读取 direction-assessment.md**,从用户需求中提炼测试目标,目标决定出题策略
2. **出题前必须先读取 goal-decomposition.md**,按目标逆向拆解知识组件→证据映射→题目设计
3. **出题前必须先读取 quiz-format.md**,确保生成的 JSON 格式正确
4. **归因前必须先读取 attribution-guide.md 和 attribution-prompt.md**,前者是标签定义权威来源,后者是执行编排

### 环境约束

5. **跨平台执行**：所有命令按平台差异执行，平台检测流程见 `setup/env-check.md`，端口管理见 `setup/port-clean.md`，前端启动见 `setup/frontend-launch.md`
6. **两阶段分离**:Vite 启动后命令立即返回,不会阻塞到答题完成。用户需要主动回来触发阶段2
7. **档案驱动**:所有数据写入用户指定的档案目录 `{ARCHIVE_PATH}/sessions/{date}/`,由 Git 自动维护版本
8. **端口管理**：每次启动前主动清理 5199 端口，详见 `setup/port-clean.md`；用户可通过触发词“关闭面试”主动关闭端口

### 数据约束

8. **sessionId 格式**:统一用 `if-{YYYYMMDD-HHmmss}`,保证唯一性
9. **JSON 安全**:题库 JSON 的 stem/options 字段中,禁止使用中文引号(""),必须用书名号(「」)或英文引号替代
10. **自动判断阶段**:如果用户回来时已有 result 文件,自动进入阶段2,无需用户明确说"答题完成"

### 归因质量约束

11. **叙事精度评估**:归因时必须执行叙事精度评估(详见 attribution-guide.md),HTML 报告必须包含叙事风险标注
12. **版本意识评估**:归因时必须评估版本意识(详见 attribution-guide.md「版本意识评估」)。declining 知识点即使用户答案正确但未提及 successor,也标 `versionAwareness: "absent"`
13. **题目质量审计**:归因时不只审用户,也审题目(详见 attribution-guide.md「题目质量审计」),用户的薄弱可能部分归因于出题方式
14. **lifecycle 字段必填**:出题时每道题必须标注 `lifecycle` 字段(详见 phase1-knowledge-lifecycle.md),declining 知识点的 stem 必须带版本上下文
15. **归因后诊断映射**:归因完成后必须为每个 actionPlan 条目标注 `diagnosisType` 和 `recommendedAction`(详见 attribution-guide.md「归因后诊断映射」),Step 4 根据 diagnosisType 差异化后续提议
