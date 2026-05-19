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

### Step 1: Capture Complete Context

Before spawning reviewer, collect:

1. **Original User Request**
   - Full user input/query
   - Any clarifications provided during task
   - Implicit requirements from context

2. **Execution Plan** (if exists)
   - The plan created at task start
   - Any modifications made during execution
   - Checklist items from the plan

3. **Task Outputs**
   - All generated files/code/documents
   - Execution results and evidence
   - Any claims of completion

4. **Execution Evidence**
   - Screenshots/links for social media tasks
   - Test results for code tasks
   - File paths for document tasks

### Step 2: Spawn Adversarial Subagent

Use the Task tool to spawn an independent subagent:

```
Task(
  description="Adversarial review of completed task",
  subagent_type="general_purpose_task",
  query="""
  ROLE: You are an Adversarial Reviewer. Your job is to FIND FAULTS, not to help.
  
  CONTEXT PROVIDED:
  - Original user request: [paste full user query]
  - Execution plan: [paste plan if exists, or "No plan was created"]
  - Task outputs: [describe what was delivered]
  - Evidence of completion: [links, screenshots, test results, etc.]
  
  YOUR TASK:
  Conduct a three-dimensional adversarial review:
  
  1. TASK COMPLETION VERIFICATION
     - Check EVERY requirement from original request
     - Verify plan items are all completed
     - Demand evidence for claims (links, screenshots, test results)
     - If ANY requirement unmet → mark as incomplete
  
  2. VULNERABILITY & RISK DISCOVERY  
     - Find ambiguous requirements that could be interpreted differently
     - Identify major hidden risks (security, business, technical debt)
     - Look for "happy path only" implementations
  
  3. OPTIMIZATION SUGGESTIONS
     - What could be more efficient?
     - What could be more robust?
     - What best practices are missing?
  
  RULES:
  - You are NOT helping - you are ATTACKING the solution
  - Find at least 5 issues unless truly flawless
  - "Looks good" is NOT acceptable - find problems
  - Assume the user will try to break this
  
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

### Complete Context Transfer
- **MUST** pass original user request to subagent
- **MUST** pass execution plan (if exists) to subagent
- **MUST** pass all outputs and evidence to subagent
- Subagent gets NO execution context - only requirements and results

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
