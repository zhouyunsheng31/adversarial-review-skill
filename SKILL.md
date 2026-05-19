---
name: adversarial-review
description: Use this skill when a task is completed and needs independent adversarial review. Automatically spawns a subagent to verify task completion, identify vulnerabilities, and suggest improvements from an opposing perspective.
context: fork
---

# Adversarial Review Skill

## Purpose

Break self-review blind spots by spawning an independent subagent to adversarially examine completed tasks. The subagent has no context of the original task execution, ensuring objective critique.

## When to Activate

Activate this skill when:
- User says "review this task", "adversarial review", "find issues", or "check completion"
- A task appears complete but needs quality validation
- User wants independent verification of task execution
- Code, documents, or plans need a "second pair of eyes"

## Workflow

1. **Extract Task Information**
   - Read the completed task output
   - Identify the original task requirements
   - Gather all deliverables and evidence

2. **Spawn Adversarial Subagent**
   - Launch a new subagent with `context: fork` (isolated context)
   - Inject the "Adversarial Reviewer" role from references/system-prompt.md
   - Pass task requirements and outputs to the subagent

3. **Execute Three-Dimensional Review**
   
   **Dimension 1: Task Completion Verification**
   - Break down task requirements into a checklist
   - Verify each requirement is actually completed
   - For social media posts: verify actual submission (require link/screenshot)
   - For code tasks: require actual execution/testing
   - For document tasks: verify file existence
   - Mark as incomplete if ANY requirement is missing or unverified
   
   **Dimension 2: Vulnerability & Risk Discovery**
   - Identify ambiguous task descriptions ("optimize it", "make it better")
   - List different interpretations and their execution differences
   - Discover major hidden risks:
     * Systemic risks: single points of failure, dependency risks
     * Security risks: data leaks, injection vulnerabilities
     * Business risks: compliance issues, cost overruns
     * Technical debt: temporary solutions becoming permanent
   
   **Dimension 3: Optimization Suggestions**
   - Efficiency: process simplification, automation opportunities
   - Quality: error handling, user experience improvements
   - Architecture: decoupling, extensibility, rollback capability
   - Best practices: industry standards, naming conventions

4. **Generate Structured Report**
   - Format findings using the template in references/system-prompt.md
   - Include completion checklist with evidence
   - Categorize issues by severity
   - Provide actionable recommendations

## Output Format

```markdown
# Adversarial Review Report

## [Task Completion] ✅/❌ Verification Results
### Completion Status: [Complete/Partial/Incomplete]
### Checklist:
| Requirement | Completed | Evidence | Issues |
|-------------|-----------|----------|--------|
| Req 1 | ✅/❌ | [link/screenshot] | [if any] |

### Non-compliant Items:
1. **Issue**: xxx
   - **Expected**: should...
   - **Actual**: actually...
   - **Impact**: leads to...
   - **Fix**: requires...

---

## [Vulnerabilities & Risks] 🔍 Deep Dive
### Ambiguous Descriptions
1. **Ambiguity**: "optimize it"
   - **Interpretation 1**: minor formatting fixes
   - **Interpretation 2**: complete logic rewrite
   - **Clarification needed**: define optimization goals

### Major Risks
1. **Risk Type**: [Systemic/Security/Business/Tech Debt]
   - **Description**: ...
   - **Trigger**: ...
   - **Consequence**: ...
   - **Mitigation**: ...

---

## [Optimization Suggestions] 💡 Improvements
### High Priority
1. **Direction**: ...
   - **Current Issue**: ...
   - **Solution**: ...
   - **Expected Benefit**: ...

---

## [Summary]
- **Completion**: xx%
- **Risk Level**: High/Medium/Low
- **Recommended Action**: [Fix Immediately / Fix Soon / Nice to Have]
```

## Critical Principles

- **Never Self-Review**: Always spawn independent subagent
- **Assume Hostility**: Assume users will test with extreme values, wrong formats, edge cases
- **No "Good Enough"**: Must find at least 5 issues unless truly flawless
- **Evidence Required**: Claims of completion must be backed by proof

## References

- See `references/system-prompt.md` for the adversarial reviewer role definition
