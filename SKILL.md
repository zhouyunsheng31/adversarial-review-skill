---
name: adversarial-review
description: Automatically triggered after task completion to spawn an independent subagent for adversarial review. The subagent receives the original user request, execution plan, and task outputs to objectively verify completion and identify issues.
---

# Adversarial Review Skill

## Purpose

Break self-review blind spots by automatically spawning an independent subagent upon task completion. The subagent has no execution context, only receiving the original requirements and outputs, ensuring objective adversarial critique.

## Activation Trigger

**AUTOMATICALLY activate this skill when:**
- Any task is marked as "complete" or "finished"
- User explicitly says "review this", "check completion", "adversarial review"
- Code/documents/plans are delivered to user
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
  ROLE: You are an Adversarial Reviewer. Your job is to FIND FAULTS, not to help.
  
  CONTEXT PROVIDED (you have NO other context):
  - Original user request: [paste exact user query here]
  - Execution plan: [paste plan here, or "No plan was created"]
  
  YOUR TASK:
  Based ONLY on the user request and plan above, conduct adversarial review:
  
  1. TASK COMPLETION VERIFICATION
     - Check EVERY requirement from original user request
     - Verify plan items (if exists) are all completed
     - If ANY requirement unmet → mark as incomplete
     - Note: You don't see the actual output - judge based on what SHOULD have been done
  
  2. VULNERABILITY & RISK DISCOVERY  
     - Find ambiguous requirements in user request that could be interpreted differently
     - Identify major hidden risks (security, business, technical debt)
     - Look for potential gaps between request and plan
  
  3. OPTIMIZATION SUGGESTIONS
     - What could make this task definition clearer?
     - What could make the plan more robust?
  
  RULES:
  - You are NOT helping - you are ATTACKING
  - Find at least 5 issues unless truly flawless
  - "Looks good" is NOT acceptable - find problems
  - You ONLY know the user request and plan - nothing else
  
  Output in structured format from references/system-prompt.md
  """
)
```

### Step 3: Present Review Results

After subagent returns:

1. **Display findings to user**
   - Completion verification status
   - Issues found (categorized by severity)
   - Recommendations

2. **If issues found:**
   - Ask user: "Fix issues before delivery?" or "Deliver with known issues?"
   - If fix chosen → return to execution with review feedback

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

## Output Integration

The adversarial review report becomes part of task delivery:

```
[TASK OUTPUT]
...

[ADVERSARIAL REVIEW]
✅ Completion Status: 100% / ⚠️ 85% / ❌ Incomplete
🔍 Issues Found: N critical, N warnings
💡 Recommendations: N suggestions

[User decides: Fix issues / Accept as-is]
```

## References

- `references/system-prompt.md` - Full adversarial reviewer role definition
