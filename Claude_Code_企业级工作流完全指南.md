# Claude Code 企业级工作流完全指南

> **版本**: 2026.05.16 | **适用**: 专业开发团队 | **Skills**: 91 | **架构**: BMAD + Superpowers 双引擎

---

## 目录

1. [核心概念：为什么是双引擎](#一核心概念为什么是双引擎)
2. [总架构图](#二总架构图)
3. [引擎一：BMAD —— 角色分离（42 个技能）](#三引擎一bmad--角色分离42-个技能)
4. [引擎二：Superpowers —— 工程纪律（14 个技能）](#四引擎二superpowers--工程纪律14-个技能)
5. [质量门禁（10 个技能）](#五质量门禁10-个技能)
6. [BMAD vs Superpowers：分工边界](#六bmad-vs-superpowers分工边界)
7. [全场景标准工作流](#七全场景标准工作流)
8. [同类框架深度对比](#八同类框架深度对比)
9. [快速决策速查表](#九快速决策速查表)
10. [常见问题](#十常见问题)
11. [附录：已安装 Skills 全清单](#附录已安装-skills-全清单)

---

## 一、核心概念：为什么是双引擎

```
问题：一个 Agent 既写需求 → 又设计架构 → 又写代码 → 又审查自己 = 盲区

解决：BMAD 模拟 6 个角色，不同角色有不同的思维模式
      Superpowers 用 TDD/Worktree/调试 确保执行纪律

BMAD      = 做正确的事（角色分离，PM≠Architect≠Dev≠QA）
Superpowers = 正确地做事（TDD 铁律、Worktree 隔离、结构化调试）
```

| 引擎 | 技能数 | 核心职责 | 关键词 |
|------|--------|---------|--------|
| **BMAD** | 42 | 角色分离、需求→PRD→架构→Epics→Stories→审查 | PM, Architect, Dev, QA |
| **Superpowers** | 14 | TDD、Worktree、调试、验证、代码审查 | RED→GREEN→REFACTOR |
| **质量门禁** | 10 | 安全、韧性、反幻觉、根因、设计 | Vibe Guard, fact-check |
| **辅助** | 25 | 文档、设计、调研、自动化 | Anthropic, feiskyer |

---

## 二、总架构图

```
                         你的需求 / 一句话
                            │
┌───────────────────────────┼───────────────────────────┐
│              引擎一：BMAD（角色分离 — 做正确的事）       │
│                                                       │
│  Mary          John           Winston                 │
│  业务分析师  →  产品经理    →  架构师                   │
│  市场/竞争      PRD 文档      系统设计                  │
│                                                       │
│  Sally         Amelia         审查层                  │
│  UX 设计师  →  高级开发    →  Blind Hunter             │
│  交互设计      代码实现       Edge Case Hunter         │
│                              Acceptance Auditor       │
│                                                       │
│  产出：PRD + 架构文档 + UX 设计 + Epics + Stories       │
└───────────────────────────┬───────────────────────────┘
                            │
┌───────────────────────────┼───────────────────────────┐
│           引擎二：Superpowers（工程纪律 — 正确地做事）    │
│                                                       │
│  brainstorming → writing-plans → git-worktrees        │
│  需求二次确认      拆 2-5min task    隔离环境           │
│                                                       │
│  TDD (RED→GREEN→REFACTOR) → systematic-debugging      │
│  铁律测试先行              复现→诊断→修复→验证          │
│                                                       │
│  code-review (5并行) → verification → finishing        │
│  多角度审查              完工验证       分支收尾        │
│                                                       │
│  产出：TDD 验证代码 + 规范化 commit + 审查报告          │
└───────────────────────────┬───────────────────────────┘
                            │
┌───────────────────────────┼───────────────────────────┐
│                  质量门禁（三层防线）                     │
│                                                       │
│  Vibe Guard        fact-check        find-cause       │
│  安全+韧性+可读     反幻觉验证         根因证据链        │
│                                                       │
│  design-anti-patterns  commit-staged  verification     │
│  禁止 AI 审美         规范化 commit   完工前强制验证     │
└───────────────────────────────────────────────────────┘
```

---

## 三、引擎一：BMAD —— 角色分离（42 个技能）

### 3.1 六个角色

BMAD 的核心思想：**不同角色有不同的思维盲区，让他们互相审查。**

| 角色 | 技能名 | 职责 | 怎么叫 |
|------|--------|------|--------|
| **Mary** 业务分析师 | `bmad-agent-analyst` | 战略分析、竞品调研、需求挖掘 | "叫 Mary 来做业务分析" |
| **John** 产品经理 | `bmad-agent-pm` | PRD 编写、需求发现、验收标准 | "叫 John 来写 PRD" |
| **Winston** 架构师 | `bmad-agent-architect` | 系统架构、技术选型、方案设计 | "叫 Winston 来设计架构" |
| **Sally** UX 设计师 | `bmad-agent-ux-designer` | UX 模式、交互设计、原型 | "叫 Sally 来做 UX" |
| **Amelia** 高级开发 | `bmad-agent-dev` | Story 执行、代码实现 | "叫 Amelia 来实现" |
| **Paige** 技术文档 | `bmad-agent-tech-writer` | 文档编写、知识管理 | "叫 Paige 来写文档" |

### 3.2 BMAD 标准开发流程（12 步）

这是从零开始做大功能的完整路径。每一步都有对应的 slash 命令。

```
第 1 步  /bmad-product-brief
         ├─ 做什么：定义产品简报（问题、目标用户、MVP 范围）
         ├─ 产出：product-brief.md
         └─ 耗时：15-30 min

第 2 步  /bmad-create-prd
         ├─ 做什么：John(PM) 主导编写完整 PRD（用户故事、验收条件、指标、风险）
         ├─ 产出：PRD.md
         └─ 耗时：30-60 min

第 3 步  /bmad-validate-prd
         ├─ 做什么：验证 PRD 是否符合标准（完整性、清晰度、可测试性）
         ├─ 产出：验证报告
         └─ 耗时：5-10 min

第 4 步  /bmad-create-architecture
         ├─ 做什么：Winston(Architect) 设计技术方案（架构图、数据流、技术选型）
         ├─ 产出：architecture.md
         └─ 耗时：30-60 min

第 5 步  /bmad-create-ux-design
         ├─ 做什么：Sally(UX) 设计交互模式、页面流程、组件规范
         ├─ 产出：ux-design.md
         └─ 耗时：20-40 min

第 6 步  /bmad-check-implementation-readiness
         ├─ 做什么：验证 PRD + 架构 + UX 三者一致，可以开始实现
         ├─ 产出：就绪确认 / 缺口报告
         └─ 耗时：5-10 min

第 7 步  /bmad-create-epics-and-stories
         ├─ 做什么：将 PRD 拆解为 Epics 和 User Stories，排优先级
         ├─ 产出：epics-and-stories.md
         └─ 耗时：20-30 min

第 8 步  /bmad-sprint-planning
         ├─ 做什么：生成 Sprint 计划，初始化状态追踪
         ├─ 产出：sprint-plan.md
         └─ 耗时：10-15 min

第 9 步  /bmad-create-story
         ├─ 做什么：为每个 Story 创建独立上下文文件（含所有相关 spec 片段）
         ├─ 产出：story-xxx.md
         └─ 耗时：3-5 min/story

第10步  /bmad-dev-story
         ├─ 做什么：Amelia(Dev) 执行 Story 实现
         ├─ 产出：代码 + 测试
         └─ 耗时：10-60 min/story

第11步  /bmad-code-review
         ├─ 做什么：3 层并行审查（Blind Hunter / Edge Case Hunter / Acceptance Auditor）
         ├─ 产出：审查报告（CRITICAL / HIGH / MEDIUM / LOW）
         └─ 耗时：5-15 min

第12步  /bmad-retrospective
         ├─ 做什么：Epic 完成后回顾，提取经验教训
         ├─ 产出：retrospective.md
         └─ 耗时：10-15 min
```

### 3.3 BMAD 审查体系（3 层并行）

这是 BMAD 最核心的差异化能力——不是"看一下代码"，而是三个独立审查层：

| 审查层 | 来源 | 做法 | 发现什么问题 |
|--------|------|------|------------|
| **Blind Hunter** | `bmad-code-review` 第1层 | 不看 spec，纯看代码质量 | 命名混乱、逻辑复杂、重复代码 |
| **Edge Case Hunter** | `bmad-review-edge-case-hunter` | 遍历所有分支路径和边界条件 | null/空数组/超长输入/并发冲突 |
| **Acceptance Auditor** | `bmad-code-review` 第3层 | 对照 spec 逐条验收 | 功能遗漏、验收条件不满足 |

**示例：BMAD 审查报告**
```
🔴 CRITICAL (2)
  - [Edge Case] 退款金额为 0 时未处理 → 会导致除零异常
  - [Acceptance] 未实现"部分退款"验收条件 #3

🟠 HIGH (3)
  - [Blind] RefundService.process() 圈复杂度 15，建议拆分为 3 个方法
  - [Edge Case] 并发退款请求未加锁，可能导致重复退款

🟡 MEDIUM (4)
  - [Blind] 变量名 r、a、s 不清晰，建议改为 refund/amount/status

🟢 LOW (2)
  - 日志级别建议从 debug 改为 info
```

### 3.4 BMAD 快速模式

不是所有改动都需要走 12 步全流程：

```
/bmad-quick-dev     → 一句话描述需求/bug，直接实现（适合小改动）
/bmad-code-review   → 快速审查
```

### 3.5 BMAD 特色能力

| 技能 | 用途 | 什么时候用 |
|------|------|-----------|
| `bmad-party-mode` | 6 个角色同时圆桌讨论 | 架构决策、方案评审、头脑风暴 |
| `bmad-brainstorming` | 创意方法（苏格拉底/第一性原理/预演/红队） | 需求模糊、需要创新方案 |
| `bmad-review-adversarial-general` | 对抗性审查（挑刺模式） | 关键功能上线前 |
| `bmad-advanced-elicitation` | 深度追问（苏格拉底式） | AI 说的太浅、需要深挖 |
| `bmad-technical-research` | 技术调研 | 选型、新技术评估 |
| `bmad-domain-research` | 行业调研 | 进入新领域 |
| `bmad-market-research` | 竞品调研 | 产品规划 |
| `bmad-document-project` | 棕地项目文档化 | 接手老项目，给 AI 上下文 |
| `bmad-generate-project-context` | 生成 project-context.md | 初始化项目 |
| `bmad-prfaq` | Amazon Working Backwards | 产品概念验证 |
| `bmad-correct-course` | Sprint 中重大变更 | 需求变了、架构需要调整 |
| `bmad-qa-generate-e2e-tests` | 已有功能补 E2E 测试 | 测试债偿还 |
| `bmad-customize` | 自定义 BMAD 行为 | 团队有自己的流程偏好 |
| `bmad-distillator` | 文档压缩 | 长文档给 AI 前压缩 |
| `bmad-shard-doc` | 大文档拆分 | 一个 md 太长，按章节拆分 |
| `bmad-edit-prd` | 编辑已有 PRD | PRD 需要更新 |
| `bmad-editorial-review-prose` | 文案润色 | 文档、提示词文案 |
| `bmad-editorial-review-structure` | 结构编辑 | 文档结构混乱需要重组 |
| `bmad-index-docs` | 生成文档索引 | docs/ 目录文件太多 |
| `bmad-sprint-status` | Sprint 状态一览 | 每日站会前 |
| `bmad-checkpoint-preview` | 人工在环审查 | 重要改动需要人工确认 |
| `bmad-help` | BMAD 导航 | 不知道下一步做什么时 |

### 3.6 BMAD 工作流示例对话

```
你：我要做一个用户积分系统

Claude (bmad-product-brief)：
  先确认几个问题：
  1. 积分的来源是什么？购买/签到/评价/邀请？
  2. 积分能兑换什么？优惠券/实物/权益？
  3. 积分是否有有效期？
  ...

你：购买返积分，可兑换优惠券，积分 1 年有效

Claude (bmad-create-prd，以 John PM 角色)：
  我来编写 PRD：
  # 用户积分系统 PRD
  ## 用户故事
  - 作为用户，我购买商品后自动获得积分
  - 作为用户，我用积分兑换优惠券
  - 作为运营，我能配置积分规则
  ...

你：PRD 没问题，继续架构设计

Claude (bmad-create-architecture，以 Winston Architect 角色)：
  # 积分系统架构设计
  ## 数据模型
  - points_account(用户积分账户)
  - points_transaction(积分流水)
  - points_rule(积分规则配置)
  ## 关键决策
  - 积分计算：异步 MQ 解耦，避免阻塞订单流程
  - 并发控制：乐观锁（version 字段），积分扣减场景
  - 过期处理：定时任务 + 批量过期，避免实时扫描
  ...

你：架构 OK，拆成 Stories 并开始实现

Claude：
  /bmad-create-epics-and-stories → 产出 3 个 Epics, 12 个 Stories
  /bmad-sprint-planning → Sprint 1 含 5 个 Stories
  /bmad-create-story → 为 Story-1 创建上下文文件
  /bmad-dev-story → Amelia 实现 Story-1
  [切换到 Superpowers 引擎执行 TDD]
```

---

## 四、引擎二：Superpowers —— 工程纪律（14 个技能）

### 4.1 核心循环

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   brainstorming → writing-plans → executing-plans            │
│        ↑              ↑               ↑                     │
│   理清需求        拆 task        逐 task TDD                  │
│   2-3 方案        2-5 min/task   RED→GREEN→REFACTOR          │
│                                                              │
│   辅助：git-worktrees(隔离) / systematic-debugging(调试)       │
│   收尾：code-review → verification → finishing                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 每个技能详解

#### `/brainstorming`

**什么时候调用：** 任何创造性工作之前（BMAD 产出 PRD 后、动手写代码前）

**它做什么：**
1. 先读项目文件理解现有架构
2. 一次只问一个问题（苏格拉底式）
3. 提出 2-3 种设计方案 + trade-off 分析
4. 分段展示设计并逐段获取确认
5. 将确认后的设计保存为 spec 文档

**与 BMAD 的关系：** BMAD 的 PRD 和架构是"宏观设计"，brainstorming 是"微观确认"——在动手前做最后一次需求澄清。

#### `/writing-plans`

**输入：** BMAD 产出的 Story 或 brainstorming 产出的 spec

**产出：** tasks.md，每个 task 精确到：
- 具体文件路径
- 代码改动内容
- 验证命令（怎么证明完成了）

**示例产出：**
```markdown
### Task 1: 创建 PointsAccount 数据模型
- 文件：src/models/points-account.ts
- 内容：PointsAccount 接口（userId, balance, totalEarned, totalSpent, version）
- 验证：npx tsc --noEmit 通过

### Task 2: 写积分获取测试（RED）
- 文件：src/services/__tests__/points-service.test.ts
- 内容：测试 earnPoints(userId, amount) 后余额增加
- 验证：npx jest points-service.test.ts --watchAll=false → 确认 RED

### Task 3: 实现积分获取逻辑（GREEN）
- 文件：src/services/points-service.ts
- 内容：实现 earnPoints，含乐观锁版本控制
- 验证：npx jest points-service.test.ts --watchAll=false → GREEN
```

#### `/executing-plans`

**对每个 task 执行：**
```
RED     → 先写测试，确认它失败
GREEN   → 最小代码让测试通过（不多写一行）
REFACTOR → 清理代码结构，保持测试绿
VERIFY  → 跑验证命令，检查退出码和完整输出
DONE    → 标记完成，移到下一个 task
```

**执行模式选择：**

| 模式 | 适用 | 行为 |
|------|------|------|
| 内联执行 | 简单任务 | 当前 Agent 顺序执行所有 task |
| `/subagent-driven-development` | 复杂多任务 | 每 task 独立子 Agent + 独立 worktree |

#### `/subagent-driven-development`

**适用场景：** BMAD 拆出的 Stories 包含 3+ 个独立 task 时

```
主 Agent 读 tasks.md
  ├─ 识别可并行 task
  ├─ 为每个 task 创建独立 git worktree
  ├─ 派发子 Agent 到各 worktree
  │    ├─ Agent A: API 模块（worktree A）
  │    ├─ Agent B: 前端组件（worktree B）
  │    └─ Agent C: 数据库迁移（worktree C）
  ├─ 子 Agent 完成后自动审查
  └─ 不通过的回退重做
```

#### `/test-driven-development`

**铁律级强制 TDD。** 不要想着"先写代码再补测试"。

```
RED      → 写失败的测试
GREEN    → 最少量代码通过测试
REFACTOR → 清理代码结构
```

#### `/systematic-debugging`

**4 阶段结构化调试（绝不乱试）：**

| 阶段 | 动作 | 产出 |
|------|------|------|
| 1. 复现 | 确认 bug 存在，写复现步骤 | 可复现的测试用例 |
| 2. 诊断 | 二分法定位根因，提假设→验证 | 根因分析 |
| 3. 修复 | 最小改动修复根因 | fix + 测试 |
| 4. 验证 | 确认修复不引入新问题 | 全量测试通过 |

**反模式（这个 skill 会防止）：**
- ❌ "试试改这里"
- ❌ "加上 try-catch"
- ❌ "重启一下看看"
- ✅ 先复现 → 诊断 → 修复 → 验证

#### `/using-git-worktrees`

自动创建 git worktree 隔离开发环境。配合 BMAD 的 Story 执行，每个 Story 一个独立 worktree。

#### `/requesting-code-review`

启动 5 个并行 Agent 从不同角度审查：
1. 规范合规 — 是否符合 spec/设计方案
2. 代码质量 — 命名、结构、重复、复杂度
3. 安全审计 — OWASP 常见漏洞
4. 性能评估 — 潜在瓶颈、不合理算法
5. 测试覆盖 — 测试是否充分、边界是否覆盖

#### `/verification-before-completion`

**强制行为：**
- 必须跑 fresh 验证命令（不能拿缓存结果）
- 必须确认退出码为 0
- 必须看完整输出、不只看摘要
- 不能口头说"完成了"，必须有输出证据

#### `/finishing-a-development-branch`

结构化收尾选项：
1. 合并到主分支（通过 PR）
2. 保留为独立分支
3. 丢弃（实验方案不可行）
4. 提取部分改动

---

## 五、质量门禁（10 个技能）

### 5.1 六层防线

| 防线 | 技能 | 检查什么 | 不通过后果 |
|------|------|---------|----------|
| 1. 安全 | `vibe-secure` | 硬编码密钥、注入漏洞、认证缺陷、供应链风险 | 必须修复 |
| 2. 韧性 | `vibe-check` | N+1 查询、边界情况、错误处理、资源泄漏 | 必须修复 |
| 3. 可读性 | `vibe-explain` | 认知债务评分（1-10），AI 代码能否看懂 | ≥7 分重写 |
| 4. 反幻觉 | `fact-check` | WebFetch 验证 AI 声明，拒绝记忆编造 | 不让标记完成 |
| 5. 根因 | `find-cause` | 证据链驱动的根因调查，修复前先验证 | 不让修复 |
| 6. 设计 | `design-anti-patterns` | 禁止 AI 审美（渐变/毛玻璃/大圆角） | 自动激活 |

### 5.2 Vibe Guard 三件套详解

```bash
# 默认：检查 git diff 中未提交的改动（3 遍全量）
/vibe-guard

# 全仓库扫描
/vibe-guard --full

# 快速模式：只检查严重安全+韧性，约 10 秒
/vibe-guard --quick
```

#### `/vibe-secure` — AI 代码常见安全问题

- 硬编码密钥和 Token
- SQL/NoSQL 注入面
- XSS 漏洞
- 缺失的认证/授权检查
- 不安全的默认配置
- 已知供应链风险依赖

#### `/vibe-check` — AI 代码常见生产事故

- N+1 查询问题
- 缺失的错误处理
- 边界情况未处理（空值、空数组、超长输入）
- 资源泄漏（未关闭的连接、文件句柄）
- 数据完整性风险（并发写入、事务缺失）

#### `/vibe-explain` — 认知债务地图

找出你看不懂的 AI 代码块，生成自然语言解释，并打分。

| 分数 | 含义 | 建议 |
|------|------|------|
| 1-3 | 可读性好 | OK |
| 4-6 | 有些晦涩 | 可以接受 |
| 7-10 | 严重认知债务 | **重写** |

### 5.3 fact-check — 反幻觉验证

**解决的 AI 核心问题：** Claude 会自信地编造 API 端点、不存在的库、错误的函数签名。

```
/fact-check "Y 库是否真的支持 X 功能"
→ 并行启动多个验证 Agent
→ 每个 Agent 必须用 WebFetch/WebSearch（不允许凭记忆回答）
→ 输出：VERIFIED / REFUTED / INCONCLUSIVE
```

### 5.4 find-cause — 根因证据链

**解决的 AI 核心问题：** AI 正确诊断了 bug，但修复到了错误的地方。

```
/find-cause "为什么登录接口返回 500"
→ 要求可复现的证明
→ 证据链：日志片段 → 代码路径 → 根因 → 验证方案
```

### 5.5 design-anti-patterns — 禁止 AI 审美

自动在生成 UI 代码时激活，禁止：
- ❌ 柔光渐变背景
- ❌ 毛玻璃效果（glassmorphism）
- ❌ 超大圆角
- ❌ 仪表盘里放 Hero Section
- ❌ 装饰性文案

遵循 Linear / Raycast / Stripe / GitHub 的设计哲学：功能性、诚实、人工设计感。

---

## 六、BMAD vs Superpowers：分工边界

| 维度 | BMAD | Superpowers |
|------|------|-------------|
| **核心哲学** | 角色分离，模拟真实团队 | 工程纪律，TDD 铁律 |
| **需求** | product-brief → PRD(John) → 验收标准 | brainstorming（二次确认） |
| **架构** | create-architecture(Winston) | — |
| **UX** | create-ux-design(Sally) | — |
| **拆解** | Epics → Stories → Sprint | writing-plans (2-5 min tasks) |
| **实现** | dev-story(Amelia) | executing-plans + TDD |
| **环境隔离** | — | git-worktrees |
| **调试** | — | systematic-debugging (4阶段) |
| **审查** | 3 层并行（Blind/Edge/Acceptance） | 5 Agent 并行 |
| **反幻觉** | — | fact-check（独立来源验证） |
| **安全韧性** | — | Vibe Guard |
| **Sprint 管理** | ✅ sprint-planning/status | — |
| **回顾** | ✅ retrospective | reflection |
| **多角色讨论** | ✅ party-mode | — |
| **文档化** | ✅ document-project | — |

**核心规则：**
- BMAD 管 **"做什么"** — PRD、架构、UX、Stories、Sprint
- Superpowers 管 **"怎么做"** — TDD、Worktree、调试、验证
- 两者共管 **"审查"** — BMAD 3 层 + Superpowers 5 Agent = 8 个角度

---

## 七、全场景标准工作流

### 场景 1：从零开始新项目 🏗️

```
第 1 步  /bmad-product-brief              → 产品简报
第 2 步  /bmad-create-prd                 → John(PM) 写 PRD
第 3 步  /bmad-validate-prd               → 验证 PRD 完整性
第 4 步  /bmad-create-architecture         → Winston 设计架构
第 5 步  /bmad-create-ux-design            → Sally 设计 UX
第 6 步  /bmad-check-implementation-readiness → 就绪检查
第 7 步  /bmad-create-epics-and-stories     → Epics + Stories
第 8 步  /bmad-sprint-planning             → Sprint 计划
第 9 步  /brainstorming                    → 二次确认需求
第10步  /writing-plans                     → 拆 task 清单
第11步  /using-git-worktrees               → 隔离环境
第12步  /executing-plans                   → 逐 task TDD 执行
第13步  /bmad-code-review                  → 3 层并行审查
第14步  /vibe-guard                        → 安全 + 韧性
第15步  /fact-check                        → 反幻觉验证
第16步  verification-before-completion      → 完工验证
第17步  /finishing-a-development-branch     → 收尾
```

### 场景 2：已有项目日常迭代 🔄

```
第 1 步  /brainstorming                    → 需求澄清（5-15 min）
第 2 步  /bmad-create-story                → 创建 Story 上下文
第 3 步  /writing-plans                    → 拆 task（5 min）
第 4 步  /using-git-worktrees              → 隔离环境
第 5 步  /executing-plans                  → 逐 task TDD
第 6 步  /vibe-guard --quick               → 快速质量检查
第 7 步  verification-before-completion     → 验证
第 8 步  /commit-staged                    → 规范化 commit
第 9 步  /finishing-a-development-branch    → 收尾
```

### 场景 3：紧急修复 Hotfix 🚨

```
第 1 步  /brainstorming                    → 快速理清（5 min）
第 2 步  /bmad-quick-dev                   → 快速实现
第 3 步  /vibe-secure --quick              → 安全检查（不能跳过！）
第 4 步  verification-before-completion     → 验证
第 5 步  /commit-staged                    → 规范化 commit
```

### 场景 4：复杂 Bug 调试 🐛

```
第 1 步  /systematic-debugging             → 阶段 1：复现
第 2 步  /find-cause                       → 根因证据链
第 3 步  /systematic-debugging             → 阶段 2：诊断
第 4 步  /test-driven-development          → RED：写失败测试
第 5 步  /test-driven-development          → GREEN：最小修复
第 6 步  /test-driven-development          → REFACTOR：清理
第 7 步  /vibe-check                       → 确认无新问题
第 8 步  verification-before-completion     → 全量验证
```

### 场景 5：大功能多人协作 👥

```
第 1 步  /bmad-create-prd                  → John 写 PRD
第 2 步  /bmad-create-architecture          → Winston 设计架构
第 3 步  /bmad-create-epics-and-stories     → 拆分 Epics
第 4 步  /bmad-sprint-planning             → Sprint 计划
第 5 步  /writing-plans                    → 细化 task
第 6 步  /subagent-driven-development      → 并行子 Agent
           ├─ Agent A: API 模块（worktree A）
           ├─ Agent B: 前端组件（worktree B）
           └─ Agent C: 数据库迁移（worktree C）
第 7 步  /bmad-code-review                 → 3 层审查
第 8 步  /vibe-guard                       → 质量检查
第 9 步  /fact-check                       → 反幻觉
第10步  /bmad-retrospective                → 回顾
第11步  /finishing-a-development-branch     → 合入
```

### 场景 6：架构评审 / 技术决策 🏛️

```
第 1 步  /bmad-agent-architect             → Winston 分析现有架构
第 2 步  /bmad-party-mode                  → 6 角色圆桌讨论
第 3 步  /bmad-technical-research          → 技术调研
第 4 步  /bmad-review-adversarial-general  → 对抗性审查
第 5 步  /bmad-review-edge-case-hunter     → 边界条件猎手
第 6 步  /bmad-advanced-elicitation         → 苏格拉底追问
```

### 场景 7：接手老项目 📋

```
第 1 步  /bmad-document-project            → 棕地项目文档化
第 2 步  /bmad-generate-project-context    → 生成 project-context.md
第 3 步  /vibe-explain                     → 认知债务评估
第 4 步  /bmad-technical-research          → 技术栈调研
第 5 步  /bmad-party-mode                  → 讨论改造方案
```

---

## 八、同类框架深度对比

### 8.1 执行层：BMAD vs Superpowers vs Spec Kit vs Joycraft

| 维度 | BMAD 🥇 | Superpowers 🥇 | Spec Kit | Joycraft |
|------|---------|---------------|----------|----------|
| **技能数** | 42 | 14 | 7（阶段命令） | 7 |
| **角色模拟** | ✅ 6 角色 | ❌ | ❌ | ❌ |
| **PRD 生成** | ✅ John(PM) | ❌ | ❌ | ✅ 轻量 |
| **架构设计** | ✅ Winston | 仅 brainstorming | constitution 约束 | ❌ |
| **UX 设计** | ✅ Sally | ❌ | ❌ | ❌ |
| **TDD 强制** | ❌（配合 Superpowers） | ✅ 铁律 | ❌ | ❌ |
| **Worktree** | ❌（配合 Superpowers） | ✅ | ❌ | ❌ |
| **调试系统** | ❌（配合 Superpowers） | ✅ 4 阶段 | ❌ | ❌ |
| **审查** | 3 层并行 | 5 Agent 并行 | ❌ | ❌ |
| **Sprint 管理** | ✅ | ❌ | ❌ | ❌ |
| **多角色讨论** | ✅ party-mode | ❌ | ❌ | ❌ |
| **安装方式** | npx | 插件市场 | 插件市场 | npm |
| **适用规模** | 1-50 人 | 1-20 人 | 1-10 人 | 个人 |

**结论：BMAD + Superpowers 是最强组合。** 单独用任何一个都有盲区：
- 光用 BMAD → 没有 TDD 纪律、没有 Worktree 隔离
- 光用 Superpowers → 没有角色分离、没有 PRD/架构/UX 设计
- 两者配合 → 全覆盖

### 8.2 规范层：BMAD vs Spec Kit vs OpenSpec

| 维度 | BMAD (PRD路径) | Spec Kit | OpenSpec |
|------|---------------|----------|----------|
| 适合 | 新项目 + 大功能 | 新项目（绿地） | 已有项目（棕地） |
| 产物 | PRD + 架构 + UX + Epics | constitution→spec→plan→tasks | proposal→design→tasks→delta |
| 重量 | 重（全流程） | 中 | 轻 |
| 角色 | ✅ 6 角色 | ❌ | ❌ |

**结论：日常迭代用 OpenSpec，大功能/新项目用 BMAD 全流程。**

---

## 九、快速决策速查表

### 这个场景 → 用什么

```
"我要从零做一个新产品"
  → bmad-product-brief → bmad-create-prd → bmad-create-architecture
  → bmad-create-epics-and-stories → brainstorming → writing-plans
  → executing-plans → bmad-code-review → vibe-guard → verification

"我要在现有项目加功能"
  → brainstorming → bmad-create-story → writing-plans → executing-plans

"写完代码了，确保质量"
  → bmad-code-review → vibe-guard → fact-check → verification-before

"遇到诡异 bug"
  → systematic-debugging → find-cause → TDD → vibe-check

"技术选型 / 调研"
  → bmad-technical-research → bmad-party-mode → deep-research

"多个模块同时开发"
  → writing-plans → subagent-driven-development → bmad-code-review

"和 AI 讨论架构方案"
  → bmad-party-mode （6 角色同时发言）

"Sprint 做完了，要回顾"
  → bmad-retrospective → bmad-sprint-status

"AI 写的代码我看不懂"
  → vibe-explain → bmad-editorial-review-structure

"担心 AI 在编造 API"
  → fact-check

"接手一个老项目"
  → bmad-document-project → bmad-generate-project-context → vibe-explain
```

### Bug 类型 → 对应技能

```
不知道 bug 在哪              → systematic-debugging（先复现）
知道 bug 但不确定根因         → find-cause
写好了测试但实现不了          → TDD（从 RED 开始）
改了这里那里坏了              → vibe-check
代码能跑但担心不安全          → vibe-secure
代码能跑但你看不懂            → vibe-explain
功能 OK 但测试不通过          → systematic-debugging + TDD
AI 声称用了某个不存在的库     → fact-check
```

---

## 十、常见问题

### Q1：91 个技能，我要全部记住吗？

不需要。按场景记几个核心的：

```
新功能：bmad-create-prd → brainstorming → writing-plans → executing-plans
日常迭代：brainstorming → writing-plans → executing-plans
Bug：systematic-debugging → find-cause → TDD
收尾：vibe-guard → verification-before → finishing-branch
```

其他技能 Claude 会在合适的时机自动建议你使用。

### Q2：BMAD 12 步是不是太慢了？

12 步全流程是给**大功能**用的。小改动只需：
- `/bmad-quick-dev`（5 分钟）
- 或者 `brainstorming → writing-plans → executing-plans`（10 分钟）

**花 1 小时在 BMAD 的 PRD + 架构上，可以省 10 小时返工。**

### Q3：BMAD 和 Superpowers 的 brainstorming 有什么区别？

- `bmad-brainstorming`：创意方法工具箱（苏格拉底/第一性原理/预演/红队）
- Superpowers `brainstorming`：需求澄清 + 方案对比（更偏工程）

两者可以互补使用。日常用 Superpowers 的，需要创新突破时用 BMAD 的。

### Q4：小改动也要走 TDD 吗？

不强制，但建议：
- 复杂逻辑、数据处理 → **严格 TDD**
- 简单 UI 改动、文案修改 → 写完跑验证命令即可

但 `verification-before-completion` 不能跳过。

### Q5：BMAD 的角色是独立的 Agent 吗？

是的。当你调用 `bmad-agent-architect` 时，它会以 Winston 的角色身份启动一个子 Agent，带着"我是架构师"的思维模式工作。调用 `bmad-party-mode` 时，6 个角色的 Agent 会同时启动并进行圆桌讨论。

### Q6：fact-check 和 find-cause 真的需要吗？Claude 不是已经很准确了？

Claude 有两类典型错误：
1. **幻觉**：自信地编造 API 端点、库函数、版本号 → `fact-check` 解决
2. **诊断错位**：正确发现了问题，但修复到了错误的地方 → `find-cause` 解决

企业级代码容错率为零，这两个技能提供的是"AI 行为保险"。

### Q7：如何创建团队共享的 CLAUDE.md？

在项目根目录放 `CLAUDE.md`：
```markdown
# 项目名称
- 技术栈：React 18 + TypeScript + Node.js + PostgreSQL
- 代码风格：遵循 .eslintrc.json 和 .prettierrc
- 测试框架：Jest + React Testing Library
- 分支策略：main ← feature/* 分支，PR 合并
- 禁止：console.log、any 类型、未处理的 Promise
- BMAD 模块：bmm
```

### Q8：BMAD 和 Spec Kit 能一起用吗？

不要同时用。规则：
- 新项目 → BMAD 全流程（如果你需要角色分离）
- 新项目 → Spec Kit（如果你更喜欢 constitution 驱动的轻量方式）
- 已有项目 → OpenSpec + BMAD（日常迭代用 OpenSpec，大功能用 BMAD）

---

## 附录：已安装 Skills 全清单

### 安装路径

```
~/.claude/skills/              ← 91 个软链接（Claude Code 加载位置）

源文件：
~/.agents/skills/superpowers/            (14 skills, git clone)
~/.agents/skills/anthropic-skills/       (16 skills, git clone)
~/.agents/skills/vibe-guard-skill/       (4 skills, git clone)
~/.agents/skills/jamie-bitflight-skills/ (8 skills, SSH git clone)
~/.agents/bmad/_bmad/                    (42 skills, npx install)
~/.claude/plugins/marketplaces/
  ├── claude-plugins-official/
  ├── feiskyer-claude-code-settings/     (5 skills)
  └── claude-hud/                        (HUD 状态栏)

npm 全局：
  @anthropic-ai/claude-code
  @fission-ai/openspec
  joycraft
```

### 完整 Skill 列表（91 个）

**BMAD（42）**
```
bmad-advanced-elicitation     bmad-agent-analyst         bmad-agent-architect
bmad-agent-dev                bmad-agent-pm              bmad-agent-tech-writer
bmad-agent-ux-designer        bmad-brainstorming         bmad-check-implementation-readiness
bmad-checkpoint-preview       bmad-code-review           bmad-correct-course
bmad-create-architecture      bmad-create-epics-and-stories  bmad-create-prd
bmad-create-story             bmad-create-ux-design      bmad-customize
bmad-dev-story                bmad-distillator           bmad-document-project
bmad-domain-research          bmad-edit-prd              bmad-editorial-review-prose
bmad-editorial-review-structure  bmad-generate-project-context  bmad-help
bmad-index-docs               bmad-market-research       bmad-party-mode
bmad-prfaq                    bmad-product-brief         bmad-qa-generate-e2e-tests
bmad-quick-dev                bmad-retrospective         bmad-review-adversarial-general
bmad-review-edge-case-hunter  bmad-shard-doc             bmad-sprint-planning
bmad-sprint-status            bmad-technical-research    bmad-validate-prd
```

**Superpowers（14）**
```
brainstorming  writing-plans  executing-plans  test-driven-development
subagent-driven-development  dispatching-parallel-agents  systematic-debugging
using-git-worktrees  requesting-code-review  receiving-code-review
verification-before-completion  finishing-a-development-branch
writing-skills  using-superpowers
```

**质量门禁（10）**
```
vibe-guard  vibe-secure  vibe-check  vibe-explain  fact-check
find-cause  design-anti-patterns  commit-staged  delegate  evaluate-sdlc-layers
```

**Anthropic 官方（16）**
```
pdf  xlsx  docx  pptx  doc-coauthoring  frontend-design  web-artifacts-builder
webapp-testing  mcp-builder  claude-api  algorithmic-art  canvas-design
brand-guidelines  theme-factory  internal-comms  slack-gif-creator  skill-creator
```

**feiskyer 工具（5）**
```
spec-kit-skill  kiro-skill  autonomous-skill  deep-research  reflection
```

**编排 + 内置（4）**
```
swarm-primitives  swarm-patterns  karpathy-guidelines  update-config
```

---

> **核心理念**：BMAD 负责"做正确的事"——6 个角色各司其职、互相审查。Superpowers 负责"正确地做事"——TDD 铁律、Worktree 隔离、结构化调试。
>
> **一句话**：`bmad-product-brief → bmad-create-prd → brainstorming → writing-plans → executing-plans → bmad-code-review → vibe-guard → verification-before`
>
> **最终统计**：91 个 Skills | BMAD 42 + Superpowers 14 + 质量 10 + 辅助 25 | 7 种场景全覆盖
