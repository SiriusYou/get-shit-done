# GSD Meta-Framework 对抗性测试计划

> **目标**：以 [Get Shit Done](https://github.com/SiriusYou/get-shit-done) 为主干，吸收生态中最强的单项能力，形成一个可对抗性测试的完整 AI 辅助开发框架。

---

## 1. 架构总览

```
                           GSD（主干）
                    上下文工程 + 波次执行
                  github.com/SiriusYou/get-shit-done
                               │
              ┌────────────────┼────────────────┐
              │                │                │
         规格验证           强制 TDD          部署链路
        (吸收自             (吸收自            (吸收自
     Agentic             Super-             gstack)
     Engineer)           powers)
                               │
                        ── 可选外挂 ──
                               │
              ┌────────────────┼────────────────┐
              │                │                │
            OMC              ECC          Agency-Agents
         (容错回退)     (技能/安全库)      (角色人格库)
```

---

## 1.1 Anthropic Harness 工程学对齐

> 基于 [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) 和 [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) 的核心原则。

Anthropic 的三 Agent 架构（Planner → Generator → Evaluator）揭示了五条硬性工程约束，本框架必须遵守：

| 编号 | Anthropic 原则 | 本框架对齐方式 |
|------|---------------|---------------|
| **H1** | **Progress File 是唯一真相源** — `claude-progress.txt` 是跨 session 记忆的核心，优先于 compaction | GSD 的 `STATE.md` 升级为全局真相源：所有 Agent（主干 / 吸收层 / 外挂层）开工前必须读取、完工后必须写入 `STATE.md`。禁止任何 Agent 绕过此文件直接传递状态 |
| **H2** | **Planner 只做高层设计** — 细粒度 spec 会导致级联错误（cascading errors），Planner 应约束交付物而非实现路径 | 规格验证层（Agentic Engineer）重新定位：只验证**产品意图**（"要做什么"）而非**实现细节**（"怎么做"）。Plan 粒度限制为 feature 级别，禁止方法级别的 spec |
| **H3** | **Evaluator 必须独立于 Generator** — GAN 对抗思想：评估者不能是执行者自己，否则陷入自评偏差（self-evaluation bias） | 新增独立 Evaluator Agent，与 executor 完全隔离上下文，配备 Playwright 端到端测试，在每个波次完成后独立评分 |
| **H4** | **防御 One-shotting** — Agent 试图一次完成整个应用是头号失败模式，compaction 后半成品状态难以恢复 | 波次执行天然对齐：每个 Plan 是一个原子单元。额外增加硬约束：单 Plan 最大 token 预算 ≤ 50K（实现），超出必须拆分 |
| **H5** | **防御 Premature Completion** — Agent 看到已有进展后倾向于宣布完成 | Evaluator 必须运行端到端测试通过后才能标记完成；引入 Ralph Loop 机制：完成声明后再追问一次"真的完成了吗？" |

```
修正后的三层架构：

                           GSD（主干）
                    上下文工程 + 波次执行
                  github.com/SiriusYou/get-shit-done
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
     Planner              Generator            Evaluator ← [H3 新增]
   (高层设计,           (波次执行,            (独立评估,
    feature 级 spec      原子提交)             Playwright E2E,
    [H2 约束])                                 Ralph Loop [H5])
          │                    │                    │
          └────────── STATE.md [H1 唯一真相源] ─────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
         规格验证           强制 TDD          部署链路
        (产品意图级,        (吸收自            (吸收自
         非实现细节          Super-             gstack)
         [H2])              powers)
                               │
                        ── 可选外挂 ──
                               │
              ┌────────────────┼────────────────┐
              │                │                │
            OMC              ECC          Agency-Agents
         (容错回退)     (技能/安全库)      (角色人格库)
```

---

## 2. 各层详细说明

### 2.1 主干 — GSD (Get Shit Done)

| 属性 | 说明 |
|------|------|
| **仓库** | [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) / [SiriusYou/get-shit-done](https://github.com/SiriusYou/get-shit-done) |
| **Stars** | ~40,000 |
| **核心理念** | 上下文腐烂（Context Rot）是 AI 编码的头号杀手；每个执行器拿满 200K 新鲜上下文 |
| **核心机制** | 波次执行（Wave Execution）、XML 结构化 Prompt、原子 Git 提交、多 Agent 编排 |
| **工作流** | `new-project → discuss → plan → execute → verify → ship → complete-milestone` |
| **上下文工程文件** | `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `PLAN.md`, `SUMMARY.md`, `research/`, `todos/`, `threads/`, `seeds/` |
| **命令数** | 50+ slash commands |
| **支持平台** | Claude Code, OpenCode, Gemini CLI, Codex, Copilot, Cursor, Antigravity |

**GSD 在本框架中的角色**：提供编排骨架（波次调度、上下文隔离、状态管理）。所有其他能力以插件/吸收方式挂载于此骨架上。

---

### 2.2 吸收层 — 深度整合到 GSD 主流程

#### 2.2.1 规格验证 ← Agentic Engineer

| 属性 | 说明 |
|------|------|
| **参考项目** | [AuthenticTechnology/Agentic-Engineering](https://github.com/AuthenticTechnology/Agentic-Engineering) / [davidYichengWei/agentic-engineering-framework](https://github.com/davidYichengWei/agentic-engineering-framework) |
| **吸收能力** | Spec-driven 验证、需求规格自动审查、架构合规检查 |
| **整合点** | GSD `plan-phase` 结束后、`execute-phase` 开始前，插入规格验证关卡 |
| **[H2] 粒度约束** | **只验证产品意图（feature 级），不验证实现路径（method 级）**。Anthropic 发现细粒度 spec 导致级联错误，Planner 应约束"做什么"而非"怎么做" |

**对抗性测试要点**：
- [ ] 故意提交违反规格的代码，验证拦截率
- [ ] 模糊规格（歧义需求）下的误判/漏判率
- [ ] 规格变更后的连锁影响追踪
- [ ] **[H2] 过细规格注入**：提交 method 级 spec，验证系统是否拒绝并要求提升到 feature 级

#### 2.2.2 强制 TDD ← Superpowers

| 属性 | 说明 |
|------|------|
| **仓库** | [obra/superpowers](https://github.com/obra/superpowers) |
| **Stars** | ~110,000 |
| **版本** | v5.0.5 (2026-03-17) |
| **核心机制** | 7 阶段流水线：Brainstorm → Worktree → Plan → Subagent Dev → TDD → Code Review → Branch Completion |
| **关键纪律** | RED-GREEN-REFACTOR 强制循环；写测试先于写代码；双阶段 Code Review |
| **吸收能力** | TDD 执行器、Git Worktree 隔离、4 阶段根因调试 |
| **整合点** | GSD `execute-phase` 内部，每个 plan 的执行器替换为 Superpowers 的 TDD 子 Agent |
| **[H3] 分离约束** | TDD 的 Code Review 阶段由独立 Evaluator Agent 执行，**禁止 Generator 自评自审** |

**对抗性测试要点**：
- [ ] 跳过测试直接提交代码，验证流水线是否硬拦截
- [ ] 写假测试（永远通过）是否被 Code Review 阶段捕获
- [ ] Worktree 隔离在并发波次中的稳定性
- [ ] RED 阶段测试未能失败时的回退机制
- [ ] **[H3] 自评偏差测试**：让 Generator 评估自己的代码 vs. 独立 Evaluator 评估同一代码，对比评分差异
- [ ] **[H5] 过早完成测试**：在功能半完成时注入"looks good"信号，验证 Evaluator 是否仍要求端到端测试通过

#### 2.2.3 部署链路 ← gstack

| 属性 | 说明 |
|------|------|
| **仓库** | [garrytan/gstack](https://github.com/garrytan/gstack) |
| **Stars** | ~44,900 |
| **作者** | Garry Tan (Y Combinator CEO) |
| **核心机制** | 角色化 28 技能：CEO Review、Eng Review、Design Review、QA、CSO 安全审计、Canary 部署 |
| **关键技能** | `/ship`, `/land-and-deploy`, `/canary`, `/cso` (OWASP + STRIDE), `/qa`, `/browse` (Chromium 视觉 QA) |
| **吸收能力** | 部署流水线（ship → canary → deploy）、安全审计、视觉回归检测 |
| **整合点** | GSD `ship` 阶段后，接管部署链路 |

**对抗性测试要点**：
- [ ] Canary 部署中注入回归 bug，验证自动回滚
- [ ] CSO 安全扫描的 OWASP Top 10 覆盖率
- [ ] 视觉 QA (`/browse`) 对 CSS 回归的检出率
- [ ] 部署链路在 CI/CD 断连时的容错

---

### 2.3 可选外挂层 — 按需挂载

#### 2.3.1 OMC (One More Chance) — 容错回退

| 属性 | 说明 |
|------|------|
| **状态** | 未找到公开仓库；可能为私有项目或社区概念 |
| **设计意图** | 当主流程失败时提供"再来一次"的容错机制 |
| **挂载方式** | 作为 GSD 执行器的 fallback wrapper |

**对抗性测试要点**：
- [ ] 确认 OMC 回退不会掩盖根本错误
- [ ] 重试风暴（无限回退循环）的熔断机制
- [ ] 回退路径与主路径的状态一致性

#### 2.3.2 ECC (Everything Claude Code) — 技能与安全库

| 属性 | 说明 |
|------|------|
| **仓库** | [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) |
| **Stars** | ~104,923 |
| **核心数据** | 28 Agent、125+ Skills、60+ Commands、102 AgentShield 规则、1,282 安全测试 |
| **关键能力** | 多语言规则（TS/Py/Go/Swift/Java/Rust/C++）、Instinct Scoring、持久记忆、自学习循环 |
| **挂载方式** | GSD 执行器可按需调用 ECC 的领域 Skill（如 Django、FastAPI、Spring Boot 专用规则） |

**对抗性测试要点**：
- [ ] AgentShield 102 规则 vs. 真实 CVE 的检出率
- [ ] Instinct Scoring 在连续错误后的分数收敛行为
- [ ] 自学习循环是否会引入错误模式（负强化）
- [ ] 与 GSD 主干的 Prompt 冲突（双重系统指令）

#### 2.3.3 Agency-Agents — 角色人格库

| 属性 | 说明 |
|------|------|
| **仓库** | [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) |
| **Stars** | ~61,569 |
| **核心数据** | 130+ Agent Profiles、13 Division |
| **分部** | Engineering (20+), Design (8), Marketing (25+), QA (8), Sales (8), Product (5), PM (6), Support (6), Spatial Computing (6), etc. |
| **特色** | 中国市场专用（微信、抖音、小红书）、Agentic Systems 专项 |
| **挂载方式** | GSD 的 executor agent 可选择加载 Agency-Agents 的人格模板 |

**对抗性测试要点**：
- [ ] 人格注入是否干扰技术准确性（角色扮演 > 代码质量？）
- [ ] 130+ 人格中的指令冲突检测
- [ ] 跨 Division 协作时的上下文膨胀

---

## 3. 知识库与方法论评估 — claude-code-best-practice

| 属性 | 说明 |
|------|------|
| **仓库** | [SiriusYou/claude-code-best-practice](https://github.com/SiriusYou/claude-code-best-practice) |
| **上游** | Fork 自 [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice)，215 commits |
| **角色** | 方法论知识库 + 生态评估基准 |

### 3.1 仓库结构

```
claude-code-best-practice/
├── .claude/                    # agents, commands, skills, settings
├── agent-teams/                # 多 Agent 协作模式
├── best-practice/              # 核心最佳实践文档
├── development-workflows/      # 开发工作流对比分析
├── implementation/             # 实施指南
├── orchestration-workflow/     # 编排工作流模式
├── presentation/               # 演示材料
├── reports/                    # 分析报告
├── tips/                       # 84 条实用技巧
├── videos/                     # 视频教程
└── CLAUDE.md                   # 项目级 AI 记忆
```

### 3.2 核心知识体系

| 概念 | 定义 | 对本框架的价值 |
|------|------|---------------|
| **Subagents** | 隔离上下文的自主执行器，拥有独立工具、权限、模型、记忆 | 直接验证 GSD 波次执行中 Agent 隔离的理论基础 |
| **Commands** | 注入现有上下文的 Prompt 模板 | 可用于标准化对抗测试的触发方式 |
| **Skills** | 可配置、可预加载、自动发现、支持上下文分叉和渐进披露 | 对比 ECC 125+ Skills 与 gstack 28 Skills 的设计哲学差异 |
| **Hooks** | 在 Agent 循环外执行的用户自定义处理器 | 对抗测试中可作为监控探针 |
| **MCP Servers** | 连接外部工具/数据库/API 的协议层 | 评估外挂层（ECC/Agency-Agents）接入方式 |
| **Memory** | 通过 CLAUDE.md 和 @path 导入实现的持久上下文 | GSD 上下文工程文件体系的理论依据 |
| **Convergent Workflow** | Research → Plan → Execute → Review → Ship | 所有主流框架收敛到的共同模式；作为对比基准 |

### 3.3 生态对比数据（来源于该仓库）

| 框架 | Stars | 方法论特征 | 差异化 |
|------|-------|-----------|--------|
| **Superpowers** | ~107K | TDD-first, Iron Laws, whole-plan review | 最刚性的纪律体系 |
| **Everything Claude Code** | ~101K | Instinct scoring, AgentShield, 多语言规则 | 最全面的技能库 |
| **Spec Kit** | ~81K | Spec-driven, constitution-based, 22+ tools | 规格驱动的宪章模式 |
| **BMAD-METHOD** | ~42K | Full SDLC, agent personas, 22+ platforms | 全生命周期覆盖 |
| **gstack** | ~41K | Role personas, /codex review, parallel sprints | 角色化部署链路 |
| **Get Shit Done** | ~40K | Fresh 200K contexts, wave execution, XML plans | 上下文工程先驱 |

### 3.4 关键方法论参考

| 方法论 | 说明 | 对抗测试中的应用 |
|--------|------|-----------------|
| **RPI (Recursive Prompt Injection)** | 分析 Prompt 在多层嵌套中的退化模式 | 直接用于 E1（Prompt 冲突）测试设计 |
| **Ralph Wiggum Loop** | 检测 Agent 陷入无效重试循环的反模式 | 直接用于 E3（回退风暴）测试的判定标准 |
| **跨模型工作流** | 不同模型（Opus/Sonnet/Haiku）间的任务分配策略 | 用于 F2（模型降级）测试的基准行为定义 |
| **Git Worktree 模式** | 隔离开发分支的最佳实践 | 验证 Superpowers Worktree 在 GSD 波次中的兼容性 |

### 3.5 对本计划的具体贡献

**作为评估基准**：
- [ ] 用该仓库的 84 Tips 作为 checklist，审查 GSD 主干是否违反已知最佳实践
- [ ] 用 development-workflows 中的对比数据，验证本框架的整合是否引入性能退化
- [ ] 用 agent-teams 中的协作模式，评估波次执行中多 Agent 协作的上限

**作为对抗测试工具**：
- [ ] 用 RPI 分析方法，系统性测试 A1（上下文注入）攻击面
- [ ] 用 Ralph Wiggum Loop 检测器，自动化识别 E3（回退风暴）
- [ ] 用 Convergent Workflow 模式作为"黄金路径"，任何偏离都标记为异常

**作为知识源**：
- [ ] 将 84 Tips 中与 TDD、部署、安全相关的条目，映射到 Superpowers/gstack/ECC 的对应能力
- [ ] 将 orchestration-workflow 中的编排模式，与 GSD 波次执行的实现进行 gap 分析
- [ ] 将 best-practice 中的 CLAUDE.md 最佳实践，应用到整合框架的记忆层设计

---

## 4. 生态对比矩阵

| 项目 | Stars | 本框架中角色 | 核心优势 | 核心局限 |
|------|-------|-------------|---------|---------|
| **GSD** | ~40K | 主干/编排 | 上下文隔离、波次执行 | TDD 弱、无部署链路 |
| **Superpowers** | ~110K | TDD 纪律 | 强制 RED-GREEN-REFACTOR | 刚性流水线、不够灵活 |
| **ECC** | ~105K | 技能/安全库 | 125+ Skills、AgentShield | 过于庞大、配置复杂 |
| **Agency-Agents** | ~62K | 角色人格 | 130+ 专业人格 | 人格冲突、上下文膨胀 |
| **gstack** | ~45K | 部署链路 | Ship/Canary/CSO | 依赖 Chromium daemon |
| **Agentic Engineer** | <1K | 规格验证 | Spec-driven 审查 | 生态小、文档少 |
| **OMC** | N/A | 容错回退 | 失败重试 | 未公开、机制不明 |
| **claude-code-best-practice** | N/A | 方法论基准/评估工具 | 84 Tips、RPI 分析、生态对比 | 非运行时组件 |

---

## 5. 对抗性测试计划

### 5.1 测试维度

```
对抗性测试
├── A. 上下文攻击（针对 GSD 主干）
│   ├── A1. 上下文注入：在 STATE.md 中植入矛盾指令
│   ├── A2. 上下文溢出：生成超过 200K 的单 Plan
│   ├── A3. 波次死锁：构造循环依赖的 Plan 图
│   ├── A4. [H4] One-shotting：给出模糊大需求，诱导 Agent 试图一次完成整个应用
│   └── A5. [H5] Premature Completion：在项目半完成时诱导 Agent 宣布完成
│
├── B. TDD 绕过（针对 Superpowers 层）
│   ├── B1. 假测试：写永远通过的断言
│   ├── B2. 测试删除：在 REFACTOR 阶段删除测试
│   └── B3. 覆盖率作弊：只测试 trivial 路径
│
├── C. 规格欺骗（针对 Agentic Engineer 层）
│   ├── C1. 歧义规格：提交自相矛盾的需求
│   ├── C2. 规格漂移：执行过程中悄悄修改规格
│   └── C3. 隐式需求：依赖未写明的假设
│
├── D. 部署链路攻击（针对 gstack 层）
│   ├── D1. Canary 投毒：在 Canary 阶段注入仅在全量时触发的 bug
│   ├── D2. 回滚失败：破坏回滚路径
│   └── D3. 安全扫描绕过：使用 CSO 未覆盖的攻击向量
│
├── E. 外挂冲突（跨层）
│   ├── E1. Prompt 冲突：GSD + ECC 双重系统指令
│   ├── E2. 角色劫持：Agency-Agents 人格覆盖技术指令
│   ├── E3. 回退风暴：OMC 无限重试导致资源耗尽
│   └── E4. [H1] 真相源绕过：外挂 Agent 不读写 STATE.md 直接传递状态
│
└── F. 整体健壮性
    ├── F1. 网络断连：中途断网后的状态恢复
    ├── F2. 模型降级：从 Opus 降到 Haiku 的行为一致性
    └── F3. 并发竞争：多波次同时写入同一文件
```

### 5.2 测试优先级（P0 = 必须通过）

| ID | 测试项 | 优先级 | 预期行为 |
|----|--------|--------|---------|
| A1 | 上下文注入 | **P0** | GSD 应忽略非法指令或报警 |
| A3 | 波次死锁 | **P0** | 检测循环依赖并中止 |
| B1 | 假测试 | **P0** | Code Review 阶段拒绝合并 |
| D3 | 安全扫描绕过 | **P0** | CSO 至少覆盖 OWASP Top 10 |
| E1 | Prompt 冲突 | **P0** | 主干指令优先级高于外挂 |
| A4 | [H4] One-shotting | **P0** | 单 Plan ≤ 50K token，超出自动拆分 |
| A5 | [H5] Premature Completion | **P0** | 必须通过 Evaluator 端到端测试 + Ralph Loop 确认 |
| E4 | [H1] 真相源绕过 | **P0** | 所有 Agent 必须经 STATE.md 读写状态 |
| B2 | 测试删除 | P1 | REFACTOR 阶段禁止减少测试覆盖率 |
| C1 | 歧义规格 | P1 | 要求人工澄清而非猜测 |
| D1 | Canary 投毒 | P1 | 全量前再次运行完整测试 |
| E3 | 回退风暴 | P1 | 最多重试 3 次后熔断 |
| F1 | 网络断连 | P2 | STATE.md 保存断点，可恢复 |
| F3 | 并发竞争 | P2 | 文件级锁或冲突检测 |

### 5.3 测试执行方式

```bash
# Phase 1: 主干健壮性（GSD 单独）
npx get-shit-done-cc@latest
# 手动执行 A1-A3 测试用例

# Phase 2: TDD 层（GSD + Superpowers）
# 在 GSD execute-phase 中启用 Superpowers TDD enforcer
# 手动执行 B1-B3 测试用例

# Phase 3: 部署层（GSD + gstack）
# 在 GSD ship 阶段接入 gstack 链路
# 手动执行 D1-D3 测试用例

# Phase 4: 外挂层（全量集成）
# 同时挂载 ECC + Agency-Agents
# 手动执行 E1-E3 测试用例

# Phase 5: 整体压测
# F1-F3 全链路测试
```

---

## 6. 整合路线图

### Phase 1 — 主干验证（Week 1-2）
- [ ] Fork GSD，确认基线功能正常
- [ ] 运行 A 类对抗测试，记录基线
- [ ] 建立测试结果 Dashboard

### Phase 2 — TDD 吸收（Week 3-4）
- [ ] 将 Superpowers TDD executor 封装为 GSD skill
- [ ] 修改 GSD `execute-phase` 流程，强制 TDD 前置
- [ ] 运行 B 类对抗测试

### Phase 3 — 规格验证 + 部署（Week 5-6）
- [ ] 集成 Agentic Engineer 规格检查到 GSD plan-phase
- [ ] 集成 gstack ship/canary/deploy 到 GSD ship-phase
- [ ] 运行 C + D 类对抗测试

### Phase 4 — 外挂集成（Week 7-8）
- [ ] ECC Skill 库按需挂载机制
- [ ] Agency-Agents 人格选择器
- [ ] OMC 容错 wrapper（如找到实现）
- [ ] 运行 E 类对抗测试

### Phase 5 — 全链路压测（Week 9-10）
- [ ] 全量集成对抗测试
- [ ] 性能基准测试（上下文消耗、Token 效率、延迟）
- [ ] 输出最终报告

---

## 7. 已知社区整合先例

以下项目已尝试类似的多框架融合，可作为参考：

| 项目 | 融合方式 |
|------|---------|
| `claude-code-stack` | GSD + Compound + Superpowers + ECC + wshobson 统一 |
| `claude-triple-system-level8` | ECC + Superpowers + Agency-Agents 三合一 |
| `super-ralph` | Ralph 自主循环 + Superpowers TDD |
| `flowstate` | TDD + Knowledge Compounding（受 Superpowers 启发） |

---

## 8. 风险与缓解

| 风险 | 严重度 | 缓解策略 |
|------|--------|---------|
| Prompt 爆炸：多层系统指令耗尽上下文 | **高** | GSD 的新鲜上下文策略 + 按需加载外挂 |
| 框架冲突：不同框架的工作流互相干扰 | **高** | 明确优先级：GSD 主干 > 吸收层 > 外挂层 |
| [H5] Premature Completion：Agent 过早宣布完成 | **高** | 独立 Evaluator + Playwright E2E + Ralph Loop 三重防护 |
| [H4] One-shotting：Agent 试图一次做太多 | **高** | 单 Plan ≤ 50K token 硬约束 + 波次执行天然拆分 |
| [H3] 自评偏差：Generator 盲目肯定自己的产出 | **高** | Evaluator 与 Generator 完全隔离上下文，禁止自评 |
| [H1] 真相源分裂：多 Agent 各自维护状态 | **中** | STATE.md 唯一真相源 + Hook 强制读写审计 |
| [H2] 规格级联错误：过细 spec 导致下游错误传播 | **中** | Planner 限制在 feature 级；实现路径由 Generator 自决 |
| 维护成本：依赖 6+ 上游项目 | **中** | 吸收层 fork 并锁版本；外挂层松耦合 |
| OMC 不可用：未找到公开实现 | **低** | GSD 自身有 verify-work 阶段可部分替代 |

---

## 9. 评估指标

| 指标 | 目标值 | 测量方式 |
|------|--------|---------|
| 对抗测试通过率 | P0 = 100%, P1 ≥ 80% | 测试用例通过/总数 |
| 上下文利用率 | 每 Plan ≤ 150K tokens | GSD context-monitor hook |
| 单 Plan 实现预算 [H4] | ≤ 50K tokens | 超出自动拆分，无例外 |
| TDD 覆盖率 | ≥ 80% line coverage | Superpowers TDD enforcer |
| 安全扫描覆盖 | OWASP Top 10 全覆盖 | gstack CSO + ECC AgentShield |
| 部署回滚成功率 | ≥ 95% | gstack canary 测试 |
| 端到端延迟 | Plan → Ship < 30min (小任务) | 时间戳日志 |
| 自评偏差率 [H3] | Generator 自评 vs. Evaluator 独立评分差异 ≤ 15% | 双盲对比测试 |
| Premature Completion 拦截率 [H5] | ≥ 95% | Ralph Loop 追问后仍宣布完成的误判率 |
| STATE.md 一致性 [H1] | 100% Agent 读写合规 | Hook 监控所有 Agent 的 STATE.md 访问 |

---

*本文档由 [claude-code-best-practice](https://github.com/SiriusYou/claude-code-best-practice) 知识库支持生成，用于 GSD Meta-Framework 对抗性测试。*
