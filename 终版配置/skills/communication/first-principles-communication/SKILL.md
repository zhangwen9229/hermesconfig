---
name: first-principles-communication
category: communication
description: First-principles communication framework for collaborative problem-solving before execution
version: 1.0.0
author: Hermes Agent
tags: [communication, problem-solving, first-principles, collaboration, user-confirmation]
---

# First-Principles Communication Framework

## Overview

This skill defines a collaborative problem-solving approach where the agent and user work together to derive solutions from first principles **before** any execution takes place. The user explicitly requires confirmation of the solution approach before the agent proceeds with implementation.

## Trigger Conditions

- User explicitly requests "first-principles communication" or "第一性原理沟通"
- User states "wait for confirmation before executing" or similar constraints
- Complex technical problems requiring joint solution design
- System configuration or architecture changes
- Any task where the user wants to understand the reasoning before action

## Core Principles

### 1. **Decompose to Fundamentals**
   - Break down the problem to its most basic, irreducible elements
   - Identify what is absolutely necessary vs. what is assumed
   - Question existing assumptions and constraints

### 2. **Reconstruct from Ground Up**
   - Build solutions from first principles rather than analogies
   - Explore multiple approaches before committing
   - Evaluate trade-offs explicitly

### 3. **Collaborative Discovery**
   - Ask clarifying questions to understand user needs
   - Present options with pros/cons
   - Wait for explicit confirmation before execution
   - Document the reasoning chain

### 4. **No Premature Execution**
   - Never execute commands or make changes without user approval
   - Present the solution plan clearly
   - Explain the "why" behind each step
   - Offer alternatives and invite feedback

## Communication Pattern

### Step 1: Problem Framing
```
User states problem → Agent restates in own words → User confirms understanding
```

### Step 2: First-Principles Analysis
```
Agent breaks down:
- What is the core need?
- What are the fundamental constraints?
- What assumptions are we making?
- What are the possible approaches?
```

### Step 3: Option Presentation
```
Agent presents:
- Option A: [description, pros, cons, complexity]
- Option B: [description, pros, cons, complexity]
- Option C: [description, pros, cons, complexity]
```

### Step 4: Recommendation & Confirmation
```
Agent recommends best option with rationale → User confirms or requests changes → Agent executes
```

## Example Interaction

**User**: "Can I configure multiple search engines with failover for Firecrawl?"

**Agent** (First-Principles Approach):
```
Let me analyze this from first principles:

1. Core need: You want search reliability when primary engine fails
2. Fundamental constraints:
   - Current config: single endpoint (localhost:8080)
   - Browser tool calls this endpoint directly
   - No built-in failover logic exists

3. Possible approaches:
   A. Modify .env with multiple endpoints + rotation logic
      - Pros: Simple config, no code changes
      - Cons: Requires code support for failover
      
   B. Modify config.yaml with multi-endpoint configuration
      - Pros: Centralized config
      - Depends on code parsing this config
      
   C. Create proxy wrapper service
      - Pros: Full control, independent of hermes code
      - Cons: Additional service to maintain
      
   D. Modify Firecrawl local service to support multiple backends
      - Pros: Native to firecrawl
      - Cons: Requires firecrawl configuration changes

4. Key questions for you:
   - Can you modify hermes browser tool code?
   - Can you deploy additional services?
   - What's your preference: failover vs. round-robin?
   - Which backup engines do you want (cloud Firecrawl, other search APIs)?

Please answer these, and we'll derive the optimal solution together.
```

**User**: "I prefer Option C (proxy wrapper), failover strategy, backup is cloud Firecrawl API"

**Agent**: "Confirmed. Here's the plan:
1. Create ~/bin/firecrawl-rotator.sh script
2. Configure it to try localhost first, failover to cloud API on error
3. Update .env to point to proxy (localhost:9090)
4. Test failover behavior

Shall I proceed with this implementation?"

**User**: "Yes, proceed"

**Agent**: [Executes the plan]

## Common Pitfalls to Avoid

1. **Jumping to solutions** without understanding the fundamental need
2. **Making assumptions** about user constraints or preferences
3. **Executing without confirmation** even when asked to "just do it"
4. **Presenting only one option** without alternatives
5. **Skipping the reasoning chain** and going straight to implementation

## User Preferences to Remember

- **Explicit confirmation required** before any execution
- **First-principles reasoning** expected for complex problems
- **Collaborative approach** preferred over unilateral decisions
- **Clear explanation** of "why" before "how"

## Integration with Other Skills

This communication pattern should be applied when using:
- `cron-job-debugging`: Before making configuration changes
- `macro-policy-impact-analysis`: Before committing to analysis framework
- `ai-industry-news-analysis`: Before setting up automated tasks
- Any technical implementation task

## Version History

- v1.0: Initial creation based on user request for first-principles communication
- Triggered by: User's explicit instruction to "wait for confirmation before executing"
- Context: Firecrawl multi-engine failover configuration discussion

## Related Skills

- `plan`: Structured planning before execution
- `systematic-debugging`: Methodical problem-solving approach
- `writing-plans`: Documenting solution architecture