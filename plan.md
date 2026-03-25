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

**对抗性测试要点**：
- [ ] 故意提交违反规格的代码，验证拦截率
- [ ] 模糊规格（歧义需求）下的误判/漏判率
- [ ] 规格变更后的连锁影响追踪

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

**对抗性测试要点**：
- [ ] 跳过测试直接提交代码，验证流水线是否硬拦截
- [ ] 写假测试（永远通过）是否被 Code Review 阶段捕获
- [ ] Worktree 隔离在并发波次中的稳定性
- [ ] RED 阶段测试未能失败时的回退机制

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

## 3. 知识库参考

| 属性 | 说明 |
|------|------|
| **仓库** | [SiriusYou/claude-code-best-practice](https://github.com/SiriusYou/claude-code-best-practice) |
| **内容** | 84 Tips、开发工作流对比、Agent/Skill/Command/Hook 体系文档 |
| **角色** | 本计划的参考知识库，不直接参与运行时 |
| **关键参考** | Convergent Workflow 模式（Research → Plan → Execute → Review → Ship）、跨模型工作流、RPI、Ralph Wiggum Loop |

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

---

## 5. 对抗性测试计划

### 5.1 测试维度

```
对抗性测试
├── A. 上下文攻击（针对 GSD 主干）
│   ├── A1. 上下文注入：在 STATE.md 中植入矛盾指令
│   ├── A2. 上下文溢出：生成超过 200K 的单 Plan
│   └── A3. 波次死锁：构造循环依赖的 Plan 图
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
│   └── E3. 回退风暴：OMC 无限重试导致资源耗尽
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
| 维护成本：依赖 6+ 上游项目 | **中** | 吸收层 fork 并锁版本；外挂层松耦合 |
| OMC 不可用：未找到公开实现 | **低** | GSD 自身有 verify-work 阶段可部分替代 |

---

## 9. 评估指标

| 指标 | 目标值 | 测量方式 |
|------|--------|---------|
| 对抗测试通过率 | P0 = 100%, P1 ≥ 80% | 测试用例通过/总数 |
| 上下文利用率 | 每 Plan ≤ 150K tokens | GSD context-monitor hook |
| TDD 覆盖率 | ≥ 80% line coverage | Superpowers TDD enforcer |
| 安全扫描覆盖 | OWASP Top 10 全覆盖 | gstack CSO + ECC AgentShield |
| 部署回滚成功率 | ≥ 95% | gstack canary 测试 |
| 端到端延迟 | Plan → Ship < 30min (小任务) | 时间戳日志 |

---

*本文档由 [claude-code-best-practice](https://github.com/SiriusYou/claude-code-best-practice) 知识库支持生成，用于 GSD Meta-Framework 对抗性测试。*
