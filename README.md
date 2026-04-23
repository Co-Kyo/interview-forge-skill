# InterviewForge Skill 出题引擎文档

> 版本:v1.2.0 | 最后更新:2026-04-22

## 概述

本目录是 InterviewForge 面试模拟系统的 **AI 出题引擎**,定义了从"用户需求"到"结构化题库 JSON"的完整流程规范,以及答题后的归因分析和报告生成流程。由 WorkBuddy Agent 读取并执行。

## 文件清单

```
skill/
├── SKILL.md                                    # 主流程编排(阶段调度 + 引用协调)
├── .gitignore                                  # 忽略 clone 的前端项目
├── LICENSE                                     # CC BY-NC 4.0
├── README.md                                   # 本文件
├── setup/                                      # 运维层:环境与部署
│   ├── env-check.md                            #   平台检测 + 依赖校验 + 项目部署
│   ├── port-clean.md                           #   端口清理 + 关闭面试服务
│   └── frontend-launch.md                     #   会话目录创建 + Vite 启动
├── references/                                 # Prompt 层:AI 思考指引
│   ├── phase-minus1-profile-setup.md          # Phase -1:档案初始化和配置
│   ├── phase0-direction-assessment.md         # Phase 0:需求提炼(目标推断模型)
│   ├── phase1-goal-decomposition.md           # Phase 1:目标逆向拆解(组件→证据→题目)
│   ├── phase1-knowledge-lifecycle.md          # Phase 1:知识生命周期管理
│   ├── phase1-quiz-format.md                  # Phase 1:题库 JSON 格式规范
│   ├── phase2-attribution-guide.md            # Phase 2:归因校验(标签体系+交叉检验)
│   └── phase2-attribution-prompt.md           # Phase 2:归因分析提示词
└── Tools/
    ├── scripts/
    │   └── render-report.cjs                   # 报告渲染脚本(JSON → HTML雷达图,CJS 格式)
    └── interview-forge-public/                 # 前端项目（.gitignore 忽略，首次启动自动 clone）
        ├── schemas/                              #   数据合同（Skill 拉取，前端定义）
        │   └── quiz.schema.json                  #     题库 JSON Schema（唯一真相源）
        ├── src/                                # Vue 3 + TypeScript 答题前端
        ├── server/                             # 数据处理
        ├── data/                               # 题目渲染配置
        ├── package.json                        # Node.js ≥ 20（marked@18）
        └── vite.config.ts                      # Vite 开发服务器
```

> `interview-forge-public/` 是从 [GitHub 开源仓库](https://github.com/Co-Kyo/interview-forge-public.git) clone 的前端项目,不纳入 skill 仓库版本控制。首次启动时 SKILL.md 会自动检测并 clone。

## 流程架构

```
┌─────────────────────────────────────────────┐
│ Phase -1:档案初始化(首次配置)            │
│                                             │
│  需要知识追踪档案? → 执行 profile-setup    │
│  profile-setup.md  → 创建档案目录结构       │
│  结束后进入常规流程                         │
└─────────────────────────────────────────────┘
         ↓
用户说"开始面试"
         ↓
┌─────────────────────────────────────────────┐
│ 前置步骤0:平台检测 + 环境检查 + 项目部署   │
│                                             │
│  setup/env-check.md                         │
│  检测 OS/Git/Node → 缺失则阻断提示安装     │
│  检查前端项目 → 未部署则自动 clone + install │
│  setup/port-clean.md                        │
│  启动前清理残留端口 5199                    │
└─────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────┐
│ Phase 1:出题 + 启动                        │
│                                             │
│  Phase 0  ──→  Phase 1  ──→  Phase 1  ─┐   │
│  需求提炼     目标拆解     生成题库    │   │
│  direction-   goal-decom-  quiz-format │   │
│  assessment   position     + KL-cycle  │   │
│                                        ▼   │
│                            Phase 1b:启动答题
│                            setup/frontend-launch.md
│                            Vite :5199      │
└─────────────────────────────────────────────┘
    │  用户答题...
    ▼  (结果保存到 sessions/{date}/)
用户说"答题完成"
    ↓
┌─────────────────────────────────────────────┐
│ Phase 2:归因 + 报告                        │
│                                             │
│  Phase 2a ──→  Phase 2b  ──→  Phase 2c    │
│  定位结果     LLM归因分析   脚本渲染报告   │
│  result-*.json attribution   render-      │
│               -prompt.md     report.js     │
│                                ↓          │
│              HTML 雷达报告 + JSON 归因结果│
└─────────────────────────────────────────────┘
```

**运维操作**:用户说"关闭面试" / "关端口" → 杀 Vite 进程(纯运维,不触发归因)

## 各文件职责

| 文件 | 阶段 | 解决什么问题 |
|------|------|------------|
| `SKILL.md` | 全局 | 主流程编排:阶段调度 + 引用协调 |
| `phase-minus1-profile-setup.md` | Phase -1 | 首次如何配置知识追踪档案路径?档案目录结构? |
| `phase0-direction-assessment.md` | Phase 0 | 用户到底要测什么?从模糊需求提炼最小核心目标 |
| `phase1-goal-decomposition.md` | Phase 1 | 怎么从目标逆向设计有依据的题目?组件→证据→题型 |
| `phase1-knowledge-lifecycle.md` | Phase 1 | 知识生命周期管理?题目如何按遗忘曲线分布? |
| `phase1-quiz-format.md` | Phase 1 | 题库 JSON 必须长什么样?格式规范 |
| `phase2-attribution-guide.md` | Phase 2 | 归因怎么做到结构化?标签体系+交叉检验 |
| `phase2-attribution-prompt.md` | Phase 2 | 对答题结果进行分析的具体提示词和方法 |
| `env-check.md` | 前置 | 平台检测 + 依赖校验 + 项目部署 |
| `port-clean.md` | 前置/运维 | 端口清理 + 关闭面试服务 |
| `frontend-launch.md` | Phase 1 | 会话目录创建 + Vite 前端启动 |
| `quiz.schema.json` | Phase 1 | 题库 JSON 结构合同（权威源），前端定义、Skill 拉取 |
| `render-report.cjs` | Phase 2c | 如何将 JSON 归因结果转化为 HTML 雷达报告？ |

## 设计原则

1. **分布式 Prompt 架构**:每个 references/ 文件独立维护,改一个不影响其他
2. **目标驱动出题**:先确定"测什么"再出题,而非随意生成
3. **逆向拆解**:测试目标 → 知识组件 → 证据类型 → 题目设计,每道题可追溯
4. **强制自检**:出题后必须执行质量检查(分布/有效性/策略对齐),不可跳过
5. **档案驱动追踪**:每个用户/会话拥有独立档案目录,支持长期知识生命周期管理
6. **前后端解耦**:LLM 出题端和答题前端仅通过 JSON 通信,各自独立迭代
7. **后处理脚本化**:报告生成从 LLM 输出拆离,由 Node.js 脚本渲染,降低 token 消耗
8. **运维逻辑外置**:平台检测、端口管理、前端启动等运维逻辑拆至 `setup/` 目录,SKILL.md 只做编排调度,与 `references/` Prompt 层平行
9. **端口管理**:启动前主动清理残留端口(`setup/port-clean.md`),用户可随时触发"关闭面试"关进程

## 跨平台支持

| 项目 | macOS / Linux | Windows |
|------|--------------|---------|
| Shell | 直接执行 | Git Bash 包裹 |
| nvm 激活 | `source ~/.nvm/nvm.sh` | Git Bash 内同左 |
| 杀端口 | `lsof -ti:{PORT} \| xargs kill -9` | `netstat + taskkill` |
| 前端项目路径 | `{SKILL_DIR}/Tools/interview-forge-public/` | 同左 |

> **硬依赖**:Git + Node.js ≥ 20。缺失时 SKILL.md 会 early fail 并提示安装方式。

## v1.2.0 包含的优化

- ✅ 档案初始化流程(Phase -1)
- ✅ 知识生命周期管理(Phase 1)
- ✅ 归因分析提示词独立化(Phase 2)
- ✅ HTML 报告脚本渲染(render-report.cjs)
- ✅ Token 消耗优化(输入→32% 减少,输出→HTML 脚本化)
- ✅ 跨平台兼容(macOS / Linux / Windows 双路命令)
- ✅ 平台检测 + 环境依赖 early fail
- ✅ 前端项目自动 clone(开源仓库 interview-forge-public)
- ✅ 端口管理(启动前主动清理 + "关闭面试"触发词)
- ✅ 目录结构重构(Tools/scripts + Tools/interview-forge-public 隔离)

## 与运行时代码的关系

`skill/` 是纯文档和脚本,被 WorkBuddy Agent 读取执行。`Tools/interview-forge-public/` 是 InterviewForge 答题前端,消费 `quiz-*.json` 输入、产出 `result-*.json` 输出。两者通过 JSON 文件解耦。

```
skill/ (AI 出题引擎)     Tools/interview-forge-public/ (答题前端)    skill/ (AI 报告生成)
  ↓                            ↓                                          ↓
PHASE 1:                    PHASE 1:                                 PHASE 2:
LLM 生成 quiz.json   →   用户答题   →   产生 result-*.json
                                              ↓
                                          render-report.cjs
                                          (Node.js 脚本)
                                              ↓
                                          HTML 雷达报告
                                              ↓
                                          (显示给用户)
```

## 版本规划

| 版本 | 方向 | 状态 |
|------|------|------|
| v1.0 | 核心流程 + 逆向拆解 + P0 验证 | ✅ 已完成 |
| v1.1 | 档案管理 + 知识生命周期 + 脚本渲染 | ✅ 已完成 |
| v1.2 | 跨平台兼容 + 目录重构 + 端口管理 + 开源仓库 | ✅ 已完成 |
| v2.0 | 外部题库接入(格式适配层) | 预留接口 |
