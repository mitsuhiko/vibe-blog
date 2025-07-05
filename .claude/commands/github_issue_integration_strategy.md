# GitHub Issue Integration Strategy for Agent Workflow

## Overview
This document outlines how GitHub issues will be integrated into the agent workflow to provide persistent state management, work tracking, and coordination between agents.

## Core Principles
1. **Issues as State Storage**: All work specifications, plans, and progress are stored in GitHub issues
2. **Labels for Agent Coordination**: Since agents don't have GitHub accounts, labels replace assignments
3. **Issue-Driven Workflow**: Every piece of work starts with and is tracked through a GitHub issue
4. **Work Resumption**: Agents can restore context by reading issue history and current state

## Labeling System

### Status Labels (Mutually Exclusive)
- `status:planned` - Work identified and ready to be picked up
- `status:in-progress` - Agent is actively working on this issue
- `status:blocked` - Work paused, waiting for dependencies
- `status:completed` - Work finished successfully
- `status:failed` - Work attempted but failed

### Type Labels
- `plan` - High-level plans and specifications
- `bug` - Issues found during implementation
- `enhancement` - Feature requests or improvements
- `task` - Specific implementation tasks
- `analysis` - Problem analysis documents
- `architecture` - Architecture design documents
- `research` - Research tasks

### Agent Labels (Who should handle this)
- `agent:software-architect` - For the main coordinator
- `agent:problem-analysis` - For requirements analysis
- `agent:architecture-design` - For system design
- `agent:task-breakdown` - For task decomposition
- `agent:detailed-planning` - For implementation planning
- `agent:implementation` - For coding tasks
- `agent:programming-lead` - For research coordination

### Phase Labels (Workflow Stage)
- `phase:requirements` - Requirements gathering phase
- `phase:design` - Architecture design phase
- `phase:planning` - Task planning phase
- `phase:implementation` - Active development phase
- `phase:testing` - Testing and validation phase
- `phase:integration` - System integration phase

### Priority Labels
- `priority:critical` - Must be done immediately
- `priority:high` - Important, do soon
- `priority:medium` - Normal priority
- `priority:low` - Can wait

## Issue Templates

### 1. Project Initialization Issue
```markdown
# Project: [Project Name]

## Requirements
[User requirements as provided]

## Status
- [ ] Problem Analysis Complete (Issue #X)
- [ ] Architecture Design Complete (Issue #X)
- [ ] Task Breakdown Complete (Issue #X)
- [ ] Implementation Plan Complete (Issue #X)
- [ ] Implementation Complete (Issue #X)

## Next Steps
[Current agent should update this]

Labels: plan, agent:software-architect, status:planned, phase:requirements
```

### 2. Problem Analysis Issue
```markdown
# Problem Analysis: [Project Name]

## Parent Issue
Implements requirements from #[parent issue number]

## Analysis Results
[Detailed problem analysis output]

## Key Decisions
[Important decisions made during analysis]

## Next Agent
Architecture Design Agent should use this analysis in #[next issue]

Labels: analysis, agent:problem-analysis, status:planned, phase:requirements
```

### 3. Task Implementation Issue
```markdown
# Task: [Task Name]

## Parent Issues
- Planning: #[planning issue]
- Architecture: #[architecture issue]

## Implementation Steps
[Detailed steps from planning agent]

## Progress
- [ ] Step 1
- [ ] Step 2
- [ ] Testing complete

## Blockers
[Any blocking issues]

Labels: task, agent:implementation, status:planned, phase:implementation
```

## Workflow Integration

### 1. Software Architect Agent
- **On Start**: Run `gh issue create` or pick up existing issue with `gh issue view NUMBER`
- **Creates**: Master project issue with overall tracking
- **Monitors**: All child issues for progress
- **Updates**: Master issue with completion status

### 2. Problem Analysis Agent
- **Reads**: Requirements from parent issue
- **Creates**: New issue with problem analysis results
- **Updates**: Parent issue with completion status
- **Labels**: Next issue with `agent:architecture-design`

### 3. Architecture Design Agent
- **Reads**: Problem analysis from linked issue
- **Creates**: Architecture specification issue
- **Updates**: Parent issue with architecture decisions
- **Labels**: Task breakdown issues

### 4. Task Breakdown Agent
- **Reads**: Architecture from linked issue
- **Creates**: Multiple task issues (one per major component)
- **Links**: Dependencies between task issues
- **Labels**: Tasks with appropriate phase and priority

### 5. Detailed Planning Agent
- **Reads**: Task issues marked for planning
- **Updates**: Task issues with detailed steps
- **Changes**: Label from `status:planned` to `status:in-progress`
- **Adds**: Checklist items for each implementation step

### 6. Implementation Agent
- **Queries**: `gh issue list --label "agent:implementation" --label "status:planned"`
- **Picks**: Highest priority task
- **Updates**: Issue with progress as work proceeds
- **Creates**: Bug issues if problems found
- **Closes**: Issue when implementation complete

## Commands for Agents

### Finding Work
```bash
# Find work for specific agent
gh issue list --label "agent:implementation" --label "status:planned"

# Find blocked work
gh issue list --label "status:blocked"

# Find bugs to fix
gh issue list --label "bug" --label "status:planned"
```

### Creating Issues
```bash
# Create new task
gh issue create --title "Implement user authentication" \
  --body "Task details..." \
  --label "task,agent:implementation,status:planned,phase:implementation,priority:high"

# Create bug report
gh issue create --title "Database connection fails on startup" \
  --body "Bug details..." \
  --label "bug,agent:implementation,status:planned,priority:critical"
```

### Updating Issues
```bash
# Claim an issue (mark as in progress)
gh issue edit NUMBER --add-label "status:in-progress" --remove-label "status:planned"

# Mark as blocked
gh issue edit NUMBER --add-label "status:blocked" --remove-label "status:in-progress"

# Complete an issue
gh issue close NUMBER
gh issue edit NUMBER --add-label "status:completed"
```

### Reading Issues
```bash
# View specific issue
gh issue view NUMBER

# View issue with comments
gh issue view NUMBER --comments

# List related issues
gh issue list --search "mentions:NUMBER"
```

## Benefits

1. **Persistent State**: All work is tracked and recoverable
2. **Clear Handoffs**: Labels indicate which agent should work on what
3. **Progress Visibility**: Status labels show current state
4. **Dependency Tracking**: Issues can reference each other
5. **Work History**: Complete audit trail of all decisions
6. **Parallel Work**: Multiple agents can work on different issues
7. **Error Recovery**: Failed work can be retried with full context