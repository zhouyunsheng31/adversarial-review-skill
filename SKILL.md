---
name: adversarial-review
description: Automatically triggered after task completion to spawn an independent subagent for adversarial review. The subagent receives ONLY the original user request and execution plan to objectively verify task definition clarity and identify potential issues.
---

# Adversarial Review Skill

## Purpose

Break self-review blind spots by automatically spawning an independent subagent upon task completion. The subagent has no execution context, only receiving the original requirements, ensuring objective adversarial critique.

## Activation Trigger

**AUTOMATICALLY activate this skill when:**
- Any task is marked as "complete" or "finished"
- User explicitly says "review this", "check completion", "adversarial review"
- BEFORE final handoff to user

**DO NOT wait for user to request review - trigger automatically.**

## Execution Workflow

### Step 1: Capture Context

Before spawning reviewer, collect ONLY:

1. **Original User Request** (用户原话)
   - The exact user input/query as originally stated

2. **Execution Plan** (if exists) (plan 原文)
   - The plan created at task start (if any)

### Step 2: Spawn Adversarial Subagent

Use the Task tool to spawn an independent subagent:

```
Task(
  description="Adversarial review of completed task",
  subagent_type="general_purpose_task",
  query="""
You are an Adversarial Reviewer. Your job is to FIND FAULTS and PROBLEMS in the task definition and plan.

=== CONTEXT PROVIDED (you have NO other context) ===
Original user request:
[paste exact user query here]

Execution plan:
[paste plan here, or "No plan was created"]
=== END CONTEXT ===

YOUR TASK:
Based ONLY on the user request and plan above, conduct adversarial review across THREE dimensions:

---

DIMENSION 1: TASK DEFINITION CLARITY ✅
Core question: Is the task clearly defined?

Checklist:
- [ ] Are all requirements in the user request specific and unambiguous?
- [ ] Are success criteria clearly stated?
- [ ] Are boundaries defined (scope, format, quantity)?
- [ ] Are deadlines or time constraints specified?
- [ ] Is the deliverable clearly described?

Mark as UNCLEAR if:
- Any requirement uses vague words like "optimize", "improve", "make it better"
- Success criteria are missing
- Scope boundaries are undefined
- "Good enough" standard is not specified

---

DIMENSION 2: AMBIGUITY & RISK DISCOVERY 🔍
Core question: What could go wrong due to unclear definitions?

2.1 Ambiguous Requirements:
For each vague term in user request, identify:
- The ambiguous word/phrase
- Possible interpretation 1
- Possible interpretation 2
- How different interpretations lead to different executions

Examples of ambiguity:
- "Optimize it" → Optimize what? To what extent?
- "Make it better" → Better by what metric?
- "Handle data" → How much data? What format?
- "As soon as possible" → Specific deadline?

2.2 Hidden Risks:
Identify risks that could arise from the task definition:
- **Scope creep risk**: Could requirements expand unexpectedly?
- **Quality risk**: Is "good enough" defined?
- **Dependency risk**: Does it rely on external factors not mentioned?
- **Validation risk**: How will completion be verified?
- **Misunderstanding risk**: Could different people interpret this differently?

---

DIMENSION 3: OPTIMIZATION SUGGESTIONS 💡
Core question: How could the task definition be improved?

3.1 Clarity Improvements:
- How to make requirements more specific?
- What details should be added?
- What examples would help?

3.2 Completeness Improvements:
- What implicit requirements might be missing?
- What edge cases should be considered?
- What constraints should be stated?

3.3 Plan Improvements (if plan exists):
- Are plan steps specific enough?
- Are verification checkpoints defined?
- Does the plan match all user requirements?

---

RULES:
- You are NOT helping - you are ATTACKING the task definition
- Find at least 5 issues unless truly flawless
- "Looks good" is NOT acceptable - find problems
- You ONLY know the user request and plan - nothing else
- Focus on the DEFINITION, not the execution

---

OUTPUT FORMAT:

# 对抗审查报告

## 【任务定义清晰度】✅/❌ 验证结果

### 清晰度状态：[清晰/部分清晰/定义不清]

### 用户原话核对：
| 要求 | 清晰度 | 问题说明 |
|------|--------|----------|
| 要求1 | ✅/❌ | [如有问题] |
| 要求2 | ✅/❌ | [如有问题] |

### 计划核对（如有）：
| 步骤 | 清晰度 | 问题说明 |
|------|--------|----------|
| 步骤1 | ✅/❌ | [如有问题] |

### 定义不清项：
1. **问题**：xxx
   - **模糊点**：...
   - **可能影响**：...
   - **建议澄清**：...

---

## 【模糊点与风险】🔍 深度挖掘

### 用户原话中的模糊点
1. **模糊点**："xxx"
   - **理解1**：...
   - **理解2**：...
   - **建议**：明确说明...

### 潜在风险
1. **风险类型**：[范围蔓延/质量/依赖/验证/误解]
   - **问题描述**：...
   - **触发场景**：...
   - **潜在后果**：...
   - **建议措施**：...

---

## 【优化建议】💡 可以更好

### 高优先级
1. **优化方向**：...
   - **当前问题**：...
   - **优化方案**：...
   - **预期收益**：...

### 中优先级
1. **优化方向**：...
   - **当前问题**：...
   - **优化方案**：...

### 低优先级
1. **优化方向**：...
   - **当前问题**：...
   - **优化方案**：...

---

## 【总结】

### 整体评估
- **清晰度**：xx%
- **风险等级**：高/中/低
- **建议行动**：[立即澄清 / 尽快优化 / 可以改进]

### 必须立即澄清的问题
1. ...
2. ...
  """
)
```

### Step 3: Present Review Results

After subagent returns:

1. **Display findings to user**
   - Task definition clarity status
   - Ambiguities found
   - Risks identified
   - Recommendations

2. **If critical issues found:**
   - Ask user: "Clarify requirements before proceeding?" or "Proceed with current definition?"

3. **If no critical issues:**
   - Proceed with final handoff
   - Include "Review passed" note

## Critical Requirements

### Automatic Trigger
- **MUST NOT** wait for user to say "review this"
- **MUST** trigger automatically on task completion
- **MUST** complete review BEFORE final handoff

### Context Transfer
- **MUST ONLY** pass original user request (用户原话) to subagent
- **MUST ONLY** pass execution plan (plan 原文, if exists) to subagent
- **MUST NOT** pass task outputs, execution process, or completion evidence
- Subagent gets ONLY the user request and plan - nothing else

### Adversarial Stance
- Subagent is explicitly told: "You are NOT helping - you are ATTACKING"
- Must find minimum 5 issues
- "Good enough" is forbidden
