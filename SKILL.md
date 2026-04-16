---
name: darwin-skill
description: "Darwin Skill (达尔文.skill): 自主优化器。**优化技能(Skill)**：评估 SKILL.md 的8维度评分体系 + hill-climbing 优化循环；**优化智能体(Agent)**：评估 Agent 核心文件的4维度诊断框架 + 系统性自审。棘轮机制 + 人在回路确保质量。触发词：'优化skill'、'skill评分'、'达尔文'、'darwin'、'优化智能体'、'优化agent'、'帮我改skill'、'agent太啰嗦'、'让agent更专业'。"
---

# Darwin Skill (达尔文.skill)

> 借鉴 Karpathy autoresearch 的自主实验循环，对 **Skills** 和 **Agents** 进行持续优化。
> 
> 核心理念：**评估 → 诊断 → 改进 → 验证 → 人类确认 → 保留或回滚 → 生成成果卡片**
> 
> GitHub: https://github.com/alchaincyf/darwin-skill

---

## CoPaw 架构适配说明

- **技能目录**：`/app/working/skill_pool/<skill>/` 或 `/app/working/workspaces/default/skills/<skill>/`
- **智能体目录**：`/app/working/workspaces/default/`
- **结果记录**：`/app/working/skill_pool/darwin-skill/results.tsv`
- **智能体结果记录**：`/app/working/skill_pool/darwin-skill/agent_results.tsv`
- **子Agent调用**：使用 `chat_with_agent` 技能进行独立评估
- **分支管理**：使用 git 进行版本控制

---

## 设计哲学

autoresearch 的精髓：
1. **单一可编辑资产** — 每次只改一个文件（SKILL.md 或 Agent 核心文件）
2. **双重评估** — 结构评分（静态分析）+ 效果验证（跑测试看输出）
3. **棘轮机制** — 只保留改进，自动回滚退步
4. **独立评分** — 评分用子agent，避免「自己改自己评」的偏差
5. **人在回路** — 每个优化完后暂停，用户确认再继续

---

# 模式一：优化技能 (Skill)

## 评估 Rubric（8维度，总分100）

### 结构维度（60分）— 静态分析

| # | 维度 | 权重 | 评分标准 |
|---|------|------|---------|
| 1 | **Frontmatter质量** | 8 | name规范、description包含做什么+何时用+触发词、≤1024字符 |
| 2 | **工作流清晰度** | 15 | 步骤明确可执行、有序号、每步有明确输入/输出 |
| 3 | **边界条件覆盖** | 10 | 处理异常情况、有fallback路径、错误恢复 |
| 4 | **检查点设计** | 7 | 关键决策前有用户确认、防止自主失控 |
| 5 | **指令具体性** | 15 | 不模糊、有具体参数/格式/示例、可直接执行 |
| 6 | **资源整合度** | 5 | references/scripts/assets引用正确、路径可达 |

### 效果维度（40分）— 需要实测

| # | 维度 | 权重 | 评分标准 |
|---|------|------|---------|
| 7 | **整体架构** | 15 | 结构层次清晰、不冗余不遗漏 |
| 8 | **实测表现** | 25 | 用测试prompt跑一遍，输出质量是否符合skill宣称的能力 |

### 评分规则
- 维度1-7：每个维度打 1-10 分，乘以权重得到该维度得分
- 维度8（实测表现）：跑2-3个测试prompt，按输出质量打1-10分
- **总分 = Σ(维度分 × 权重) / 10**，满分100
- 改进后总分必须 **严格高于** 改进前才保留

---

# 模式二：优化智能体 (Agent)

> 融合 Agent Self-Audit 诊断框架，专为优化 Agent 核心文件而设计。

## Agent 核心文件

| 文件 | 作用 | 优化目标 |
|------|------|---------|
| `PROFILE.md` | Agent 身份定义 | 清晰、独特、可记忆 |
| `SOUL.md` | Agent 灵魂/个性 | 一致、有深度、有边界 |
| `AGENTS.md` | 行为规范与记忆指南 | 高效、可预测、可维护 |
| `MEMORY.md` | 长期记忆 | 精简、有价值、不过时 |

## 评估 Rubric（4维度，总分100）

| # | 维度 | 权重 | 评分标准 |
|---|------|------|---------|
| 1 | **身份清晰度 (Identity)** | 25 | PROFILE.md：定位具体、风格可操作、边界明确 |
| 2 | **灵魂一致性 (Soul)** | 30 | SOUL.md：准则具体可执行、边界清晰、风格一致 |
| 3 | **行为可预测性 (Behavior)** | 25 | AGENTS.md：记忆系统清晰、安全规则明确、工具使用指南明确 |
| 4 | **记忆有效性 (Memory)** | 20 | MEMORY.md：保留4类有价值信息，删除可推导信息 |

### 评分规则
- 每个维度打 1-10 分，乘以权重得到该维度得分
- **总分 = Σ(维度分 × 权重) / 10**，满分100
- 改进后总分必须 **严格高于** 改进前才保留

### 详细评分标准

#### 维度1：身份清晰度 (25分)

| 分值 | 标准 |
|------|------|
| 8-10 | 名字清晰易记，定位具体（非泛泛"全能助手"），风格用3个形容词概括，边界声明明确 |
| 5-7 | 定位较具体，但风格描述模糊或边界不清晰 |
| 1-4 | 定位模糊或自相矛盾，缺乏风格定义 |

**常见问题：**
- "我是一个AI助手" → 应改为具体角色，如"代码审查专家"
- 同时说"简洁"和"详细" → 明确场景："简洁回答，详细解释"
- 无边界声明 → 添加"我不会..."声明

#### 维度2：灵魂一致性 (30分)

| 分值 | 标准 |
|------|------|
| 8-10 | 3-5条核心准则（具体可执行），有明确边界声明，风格描述一致，有连续性机制说明 |
| 5-7 | 有准则但表述抽象，或风格不够一致 |
| 1-4 | 准则抽象空洞，缺乏边界意识，风格混乱 |

**简化原则：**
> "如果模型能从上下文推断出来，就不需要写在文件里。"

#### 维度3：行为可预测性 (25分)

| 分值 | 标准 |
|------|------|
| 8-10 | 记忆系统说明清晰，安全规则明确，工具使用指南具体，有死规定（不可协商规则） |
| 5-7 | 有基本结构但细节不足，或某些部分冗余 |
| 1-4 | 结构混乱，关键部分缺失或过于冗长 |

#### 维度4：记忆有效性 (20分)

| 分值 | 标准 |
|------|------|
| 8-10 | 只保留4类高价值信息，删除过时可推导信息，格式规范，分类清晰 |
| 5-7 | 整体合理但有少量过时或可推导信息未清理 |
| 1-4 | 大量过时信息、具体代码细节、密钥密码等应删除内容 |

**应保留的4类信息：**

| 类型 | 示例 |
|------|------|
| user（用户偏好）| 代码风格、回答详细程度 |
| feedback（反馈纠错）| 用户纠正过的错误 |
| project（项目约定）| 不易重新推导的项目约定 |
| reference（外部资源）| 看板、监控面板、文档链接 |

**应删除的内容：**
- 文件结构、函数签名（可重新读代码）
- 当前任务进度（属于 task/plan）
- 临时分支名、PR号（很快过时）
- 具体代码细节（代码和提交记录更准确）
- 密钥、密码（安全风险）

---

# 通用优化流程

## Phase 0: 初始化

```
1. 确认优化范围：
   - Skill优化 → 扫描 /app/working/skill_pool/*/SKILL.md 或指定skill
   - Agent优化 → 读取工作区下的4个核心文件或指定文件
2. 创建 git 分支：auto-optimize/YYYYMMDD-HHMM 或 agent-optimize/YYYYMMDD-HHMM
3. 初始化 results.tsv（Skill）或 agent_results.tsv（Agent）
```

## Phase 0.5: 测试Prompt设计（仅 Skill 模式）

```
for each skill:
  1. 读取 SKILL.md，理解它做什么
  2. 设计2-3个测试prompt，覆盖：
     - 最典型的使用场景（happy path）
     - 一个稍复杂或有歧义的场景
  3. 保存到 skill目录/test-prompts.json
```

## Phase 1: 基线评估

**Skill 模式：**
```
# 结构评分（主agent执行）
读取 SKILL.md 全文
按维度1-7逐项打分（附简短理由）

# 效果评分（用子agent做）
对每个测试prompt，spawn子agent：
  - with_skill: 带着SKILL.md执行测试prompt
  - baseline: 不带skill执行同一prompt
对比两组输出，打维度8的分

# 汇总
计算加权总分
记录到 results.tsv
```

**Agent 模式：**
```
# 读取四个核心文件
读取 PROFILE.md / SOUL.md / AGENTS.md / MEMORY.md
按4维度逐项打分（附简短理由）
计算加权总分
记录到 agent_results.tsv
```

基线评估完成后，展示评分卡，**暂停等用户确认，再进入优化循环。**

## Phase 2: 优化循环

```
targets = [按基线分数从低到高排序]

for each target in targets:
  round = 0
  while round < MAX_ROUNDS (默认3):
    round += 1

    # Step 1: 诊断 - 找出得分最低的维度
    lowest_dim = 找到最低分维度

    # Step 2: 方案 - 生成1个具体改进方案
    plan = f"针对{lowest_dim}，改进方案：..."

    # Step 3: 确认 - 展示方案，用户确认后再执行
    if 用户拒绝:
      跳过该维度，继续下一个
      continue

    # Step 4: 执行
    修改文件
    git add + commit

    # Step 5: 验证
    新分数 = 重新评估()
    
    # Step 6: 决策
    if 新分数 > 旧分数:
      保留改动，记录"keep"
    else:
      回滚，记录"revert"
      break

  # 人类检查点
  展示改动摘要，用户确认后继续
```

## Phase 3: 汇总报告

展示优化报告：
- 优化数量、总实验次数、保留改进比例
- 分数变化表（before → after）
- 主要改进摘要

---

## results.tsv 格式（Skill）

```tsv
timestamp	commit	skill	old_score	new_score	status	dimension	note	eval_mode
2026-03-31T10:00	baseline	example-skill	-	78	baseline	-	初始评估	full_test
```

## agent_results.tsv 格式（Agent）

```tsv
timestamp	commit	file	old_score	new_score	status	dimension	note
2026-03-31T10:00	baseline	PROFILE.md	-	65	baseline	-	初始评估
2026-03-31T10:00	baseline	SOUL.md	-	70	baseline	-	初始评估
2026-03-31T10:00	baseline	AGENTS.md	-	55	baseline	-	初始评估
2026-03-31T10:00	baseline	MEMORY.md	-	45	baseline	-	初始评估
```

---

## 约束规则

1. **不改变核心功能和用途** — 只优化"怎么写"和"怎么执行"
2. **不引入新依赖** — 不添加原本没有的scripts或references文件
3. **每轮只改一个维度** — 避免多个变更导致无法归因
4. **保持文件大小合理** — 优化后文件不应超过原始大小的150%
5. **可回滚** — 所有改动在git分支上，用git revert而非reset --hard
6. **评分独立性** — 效果维度必须用子agent或至少干跑验证

## 边界条件与异常处理

### 场景1：文件不存在
```
if target_file 不存在:
  - Skill: 跳过并记录日志，继续下一个
  - Agent: 提示用户"X文件不存在，跳过该维度评估"
```

### 场景2：评分无法提升
```
if 连续3轮无提升:
  - 记录"瓶颈"标记
  - 展示当前最佳版本
  - 询问用户是否手动介入或跳过该target
```

### 场景3：git操作失败
```
if git commit 失败:
  - 记录错误日志
  - 继续内存中的优化
  - 提示用户手动处理git
```

### 场景4：用户中断
```
if 用户说"停止"或"取消":
  - 保存当前进度到 results.tsv
  - 展示已完成部分
  - 询问是否需要恢复
```

### 场景5：子agent超时
```
if 子agent执行超过 5分钟:
  - 超时标记
  - 使用主agent评估（降级方案）
  - 记录"独立评估失败"
```

### 跳过规则
- **无变化优化**：如果修改后内容完全相同，跳过该轮
- **过度优化**：单次修改超过500字，拆分为多次
- **重复触发**：同一维度的优化建议连续3次被拒绝，暂停该维度

---

## 使用方式

| 用户指令 | 模式 | 执行流程 |
|----------|------|---------|
| "优化所有skills" | Skill | Phase 0-3 完整流程 |
| "优化 xxx 这个skill" | Skill | 只对指定skill执行 Phase 0.5-2 |
| "评估所有skills的质量" | Skill | 只执行 Phase 0.5-1，不进入优化循环 |
| **"优化智能体/Agent"** | **Agent** | **读取4个核心文件，执行 Phase 0-2** |
| **"优化我的default agent"** | **Agent** | **只优化指定Agent，执行 Phase 0-2** |
| **"agent太啰嗦了"** | **Agent** | **诊断SOUL.md和AGENTS.md的冗余，执行优化** |
| **"让agent更有专业深度"** | **Agent** | **强化PROFILE.md身份定位，执行优化** |
| **"清理agent记忆"** | **Agent** | **只优化MEMORY.md，执行优化** |

---

## CoPaw 适配说明

本技能已适配 CoPaw/QwenPaw 架构：
- 路径使用 Docker 环境的标准路径
- 子agent调用使用 `chat_with_agent` 技能
- git 操作在 skill_pool 目录进行
- Agent 文件在工作区根目录

---

*融合 agent_self_audit v1.0.0 | Original darwin by alchaincyf | Adapted for CoPaw*
