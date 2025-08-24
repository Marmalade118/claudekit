---
description: Implement a validated specification by orchestrating concurrent agents
category: validation
allowed-tools: Task, Read, TodoWrite, Grep, Glob, Bash(command:*), Bash(stm:*), Bash(jq:*), Bash(which:*), Bash(test:*), Bash(echo:*)
argument-hint: "<path-to-spec-file>"
---

# Implement Specification

Implement the specification at: $ARGUMENTS

!which stm &> /dev/null && test -d .simple-task-master && echo "STM_STATUS: Available and initialized" || (which stm &> /dev/null && echo "STM_STATUS: Available but not initialized" || echo "STM_STATUS: Not installed")

## MANDATORY PRE-EXECUTION VALIDATION

Before launching ANY subagents:

### 0. Task Management System
- Check the STM_STATUS output above
- If status is "Available but not initialized", STOP and inform user to run `/spec:decompose` first
- If status is "Available and initialized", use STM for task retrieval
- If status is "Not installed", fall back to TodoWrite

### 1. Specification Readiness
- Verify spec file exists and is complete
- Check all dependencies and prerequisites are available
- STOP if spec quality is insufficient

### 2. Environment Preparation
- Verify all required tools are available
- Check for conflicting processes or locks
- Validate project state is clean (no uncommitted changes that could interfere)

### 3. Execution Plan Validation
- Break down spec into non-overlapping subagent tasks
- Identify critical path and dependencies between tasks
- Validate task assignments won't conflict

**CRITICAL: If any validation fails, STOP immediately and request clarification.**

## Implementation Process

### 1. Analyze Specification

Read the specification to extract:
- Implementation phases
- Technical components to build
- Dependencies between components
- Testing requirements
- Success criteria

### 2. Create or Load Task List

If STM is available:
```bash
# Export pending tasks to JSON for processing
stm list --status pending -f json > pending-tasks.json

# Get task count
TASK_COUNT=$(stm list --status pending -f json | jq 'length')

# For each task, extract details:
stm show <task-id> # Shows full task details including 'details' and 'validation' sections
```

Otherwise, use TodoWrite to create a comprehensive task list that includes:
- Foundation tasks (no dependencies)
- Core implementation tasks
- Integration tasks
- Testing and validation tasks
- Documentation updates

Organize tasks by:
- **Priority**: Critical path vs. parallel work
- **Dependencies**: What must complete before this task
- **Assignability**: Can multiple agents work on it

### 3. Iterative Implementation Workflow

**OPERATING PROCEDURE**: Each task follows a quality-assured implementation cycle:

#### Phase 1: Initial Implementation
For each task (process sequentially for dependent tasks, in parallel for independent ones):

**Available Specialized Agents:**
!claudekit list agents || echo "Using general-purpose agent (no specialized agents found)"

1. **Prepare Task Brief**:
   - Clear scope and boundaries
   - Expected deliverables
   - Files to modify/create
   - Quality requirements

2. **Launch Implementation Subagent**:
   - **ALWAYS use specialized subagents** from the list above when tasks match expert domains
   - Match task requirements to expert domains for optimal results  
   - Use `general-purpose` subagent only when no specialized expert fits
   
   If using STM, include task details:
   ```
   Task: "Implement [component name]"
   Prompt: |
     Task ID: [STM task ID]
     Title: [Task title from STM]
     
     Technical Details:
     [Contents of task 'details' section from STM]
     
     Validation Criteria:
     [Contents of task 'validation' section from STM]
     
     Additional Instructions:
     - Follow project code style guidelines
     - Implement complete functionality
     - Add appropriate error handling
     - Document complex logic
   ```
   
   If using TodoWrite:
   ```
   Task: "Implement [component name]"
   Prompt: Detailed implementation instructions including:
   - Specification reference
   - Technical requirements
   - Code style guidelines
   - Error handling requirements
   ```

#### Phase 2: Test Writing
After implementation completes:

**Available Testing Experts:**
!claudekit list agents | grep -i test || echo "Use general-purpose agent for test writing"

1. **Launch Testing Expert**:
   - Select the most appropriate testing expert from the list above based on project stack
   - If none shown, use general-purpose agent
   
   ```
   Task: "Write tests for [component name]"
   Subagent: [select from available testing experts above]
   Prompt: |
     Write comprehensive tests for the recently implemented [component/feature]:
     - Files to test: [list files from implementation]
     - Test requirements:
       * Unit tests for all functions/methods
       * Edge cases and error conditions
       * Integration points with other components
       * Coverage targets: aim for >80%
     
     Follow project testing conventions and patterns.
   ```

2. **Run Tests**:
   - Execute test suite for the component
   - Verify all tests pass
   - Check coverage metrics

#### Phase 3: Code Review
After tests are written and passing:

**Available Code Review Agents:**
!claudekit list agents | grep -i "code-review" || echo "Use general-purpose agent for code review"

1. **Launch Code Review Agent**:
   - Select code-review-expert if shown above, otherwise use general-purpose agent
   
   ```
   Task: "Review [component name]"
   Subagent: [select code-review-expert if available, otherwise general-purpose]
   Prompt: |
     Review the recently implemented [component/feature]:
     - Files modified: [list files from implementation]
     - Focus areas:
       * Architecture & design patterns
       * Code quality and maintainability
       * Security vulnerabilities
       * Performance considerations
       * Error handling completeness
       * Testing coverage needs
     
     Provide actionable feedback for any issues found.
   ```

2. **Analyze Review Results**:
   - Critical issues (must fix)
   - Important improvements (should fix)
   - Minor suggestions (nice to have)

#### Phase 4: Iterative Improvement
If issues are found:

**Available Specialist Agents for Fixes:**
!claudekit list agents | grep -E "typescript|react|security|performance" || echo "Use general-purpose agent for fixes"

1. **Fix Critical Issues**:
   - Launch appropriate specialist subagent from the list above if available
   - Otherwise use general-purpose agent with specific focus
   - Examples:
     * Security issues → Use security specialist if available, otherwise general-purpose
     * Performance issues → Use performance specialist if available, otherwise general-purpose
     * Type errors → Use typescript-expert if available, otherwise general-purpose
     * Test failures → Use testing specialist if available, otherwise general-purpose

2. **Re-Test After Fixes**:
   - Run test suite again to verify fixes
   - Ensure no regressions introduced

3. **Re-Review After Fixes**:
   - Run code-review-expert again on modified files
   - Continue cycle until no critical issues remain

4. **Update Task Status**:
   - If using STM: `stm update [task-id] --status done`
   - If using TodoWrite: Mark task as completed

#### Phase 5: Commit Changes
Once all critical issues are resolved and tests pass:

1. **Create Atomic Commit**:
   ```bash
   git add [relevant files]
   git commit -m "feat: [component name] - [brief description]
   
   - Implemented [key functionality]
   - Tests written and passing
   - Passed code review
   - Addresses spec requirements: [reference]
   
   🤖 Generated with Claude Code
   
   Co-Authored-By: Claude <noreply@anthropic.com>"
   ```

2. **Verify Commit**:
   ```bash
   git log -1 --stat
   ```

#### Phase 6: Progress Tracking

**Monitor Overall Progress**:
- If using STM: `stm list --status in-progress --pretty`
- If using TodoWrite: Track task completion in session
- Identify blocked tasks
- Coordinate dependencies

**Track Quality Metrics**:
- Tasks implemented: X/Y
- Tests written: X/Y
- Test coverage: X%
- Code reviews passed: X/Y
- Issues fixed: Count
- Commits created: Count

### 4. Integration & Validation

After each component cycle completes:
- Run full test suite to check for regressions
- Verify integration with other components
- Update documentation as needed
- Validate against specification requirements

### 5. Final Integration

Once all tasks complete:
1. Run full test suite
2. Validate against original specification
3. Generate implementation report
4. Update project documentation

## Error Handling

If any agent encounters issues:
1. Mark task as blocked in TodoWrite
2. Identify the specific problem
3. Either:
   - Launch a specialized agent to resolve
   - Request user intervention
   - Adjust implementation approach

## Progress Tracking

If using STM:
```bash
# View all tasks by status
stm list --pretty

# View specific status
stm list --status pending --pretty
stm list --status in-progress --pretty
stm list --status done --pretty

# Search for specific tasks
stm grep "authentication"
```

If using TodoWrite:
- ✅ Completed tasks
- 🔄 In-progress tasks
- ⏸️ Blocked tasks
- 📋 Pending tasks

## Success Criteria

Implementation is complete when:
1. All tasks are marked complete (STM: `stm list --status done` shows all tasks)
2. Tests pass for all components
3. Integration tests verify system works as specified
4. Documentation is updated
5. Code follows project conventions
6. All validation criteria from tasks are met (STM only)

## Example Usage

```
/spec:execute specs/feat-user-authentication.md
```

This will:
1. Read the user authentication specification
2. Load tasks from STM (if available) or create them in TodoWrite
3. Launch concurrent agents to build components
4. Track progress in STM or TodoWrite
5. Validate the complete implementation
6. Update task statuses as work progresses