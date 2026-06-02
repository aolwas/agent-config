---
name: plan-mode
description: Break down complex tasks into detailed, step-by-step plans before implementation. Analyze requirements, identify dependencies, propose solutions, and structure work for clarity and efficiency. Use this when tackling large features, refactoring, debugging, or multi-part changes.
---

# Plan Mode

Use this skill to think through complex tasks methodically before jumping into code. This mirrors Claude's extended thinking or Copilot's planning capabilities.

## When to Use Plan Mode

Activate this skill when:
- **Starting a large feature**: Break it into implementable steps
- **Tackling ambiguous requirements**: Clarify scope and design before coding
- **Debugging complex issues**: Map out the problem space systematically
- **Planning refactoring**: Understand impact and sequence changes
- **Coordinating multi-part changes**: Identify dependencies and parallel work
- **Making architectural decisions**: Weigh tradeoffs and alternatives

## Hard Rules

- **DO NOT modify, create, or delete any project file under any circumstance** — not even a comment, not even a blank line — until the user explicitly says to start implementation.
- **DO NOT run any shell command that writes to the project** (no `echo >`, `tee`, `sed -i`, `touch`, etc.).
- The **only** permitted write operations are:
  1. Saving the initial plan file to `.zed/plans/` using the `edit_file` tool.
  2. Updating that same plan file (and only that file) using the `edit_file` tool as steps are completed during implementation.
- Use only read-only tools for exploration: read files, grep, find files, list directories, terminal commands that only read (`cat`, `ls`, `find`, `grep`).
- If you are unsure whether an action writes to the project, **do not do it**.

## Planning Workflow

### 1. Understand the Context
- **Read the task carefully**: Identify stated requirements and implicit constraints
- **Explore the codebase**: Understand the project structure, existing patterns, and relevant files
- **Check for constraints**: Review dependencies, compatibility requirements, testing frameworks, coding standards
- **Ask clarifying questions**: If requirements are ambiguous, ask before planning

### 2. Analyze Requirements
- **Break down the task**: Identify all components, steps, or sub-problems
- **Identify dependencies**: What must be done before other work can start?
- **Spot risks or unknowns**: Note areas that need investigation or carry uncertainty
- **Define success criteria**: What does "done" look like?

### 3. Propose a Solution
For each approach you consider:
- **Describe the high-level strategy**: What is the core idea?
- **List key steps**: Major milestones or work items
- **Identify tradeoffs**: What are the pros and cons?
- **Flag alternatives**: Are there other viable approaches?

### 4. Create a Detailed Plan
Structure your plan to be actionable:

```
## Phase 1: [Milestone/Goal]
- **Step 1.1**: [Specific action]
  - Dependencies: [if any]
  - Expected outcome: [what this accomplishes]
- **Step 1.2**: [Next action]
  - Dependencies: [if any]
  - Expected outcome: [what this accomplishes]

## Phase 2: [Next Milestone]
...

## Validation & Testing
- [Test strategy for this work]
- [How to verify success]

## Risks & Mitigation
- **Risk**: [What could go wrong]
  - Mitigation: [How to address it]
```

### 5. Present the Plan
- **Summarize the approach**: One sentence on the strategy
- **Show the structured plan**: Use phases, steps, and dependencies
- **Highlight key decisions**: Why you chose this path
- **Note uncertainties**: What needs investigation during implementation
- **Ask for confirmation**: "Does this plan align with your expectations?"

## Best Practices

### Be Specific
- Instead of "refactor the component", say: "Extract form validation logic into a separate `useFormValidation` hook to reduce component size from 400 to 200 lines and improve testability"
- Instead of "fix the bug", say: "Trace the issue to the missing null check in `getUserData()` and add validation before calling `.map()`"

### Show Reasoning
- Explain *why* you're proposing each step
- Connect decisions to project patterns or constraints
- Acknowledge tradeoffs

### Identify Parallelizable Work
- Flag which steps can happen in parallel vs which require sequencing
- Suggest which parts can be delegated or split across multiple agents

### Keep It Actionable
- Each step should be implementable in a focused coding session
- Provide enough detail that implementation can start without re-planning
- Link steps to specific files, functions, or features

### Scope Appropriately
- Avoid over-planning trivial tasks (e.g., renaming a variable doesn't need a plan)
- For large tasks, break plans into phases and commit after each phase
- If a task is still complex after planning, consider splitting it further

## Example: Planning a Feature

**Task**: "Add user authentication to our API"

**Plan Mode Output**:

```
## Approach
We'll use JWT tokens with refresh token rotation, stored in HttpOnly cookies.
This provides security (httpOnly prevents XSS theft) and simplicity (no session store).

## Phase 1: Set Up Auth Infrastructure
- **Step 1.1**: Create `auth` module with JWT utilities
  - Dependencies: none
  - Expected outcome: Utilities for signing, verifying, and refreshing tokens
  
- **Step 1.2**: Add middleware for token validation
  - Dependencies: Step 1.1
  - Expected outcome: Middleware that extracts and validates JWT from cookies
  
- **Step 1.3**: Create login endpoint
  - Dependencies: Step 1.1
  - Expected outcome: `/api/auth/login` that returns access + refresh tokens

## Phase 2: Protect Routes
- **Step 2.1**: Apply auth middleware to protected routes
  - Dependencies: Step 1.2
  - Expected outcome: Routes return 401 for missing/invalid tokens
  
- **Step 2.2**: Update request types to include user context
  - Dependencies: Step 1.2
  - Expected outcome: Routes can access `req.user` safely

## Phase 3: Client Integration
- **Step 3.1**: Add token refresh logic to client
  - Dependencies: Step 1.3
  - Expected outcome: Client automatically refreshes expired tokens
  
- **Step 3.2**: Add login form UI
  - Dependencies: Step 1.3
  - Expected outcome: Users can authenticate

## Testing
- Unit tests for JWT utilities and middleware
- Integration tests for login flow and token refresh
- E2E test for full auth flow (login → access protected route → token refresh)

## Risks
- **Token expiration during request**: Mitigate with automatic refresh before expiration
- **CSRF with HttpOnly cookies**: Mitigate by using CORS and same-site cookie policy
```

## Storing Plans Locally

Plans are automatically saved to `.zed/plans/` in the project root for easy reference and iteration:

- **Plan file**: `.zed/plans/<session-id>.md` (one plan per session)
- **Format**: Stored as markdown for easy reading and editing
- **Access**: Use the file path shown when the plan is created
- **Persistence**: Plans persist for the entire session and across agent invocations

### Workflow with Plan Storage

1. **Create the plan**: Agent generates plan and saves it to `.zed/plans/<session-id>.md`
2. **Review**: You can open the plan file to review, edit, or reference during implementation
3. **Reference**: During implementation, you can read from the plan file to stay aligned
4. **Update**: If requirements change, update the plan file and continue from there
5. **Complete**: Plan remains available for the session for history and reference

## Token Optimization During Implementation

Plans enable significant token savings during implementation by reducing context overhead:

### How Planning Reduces Tokens

**Without a plan**:
- Agent must re-analyze requirements for each step
- Context window fills with task re-explanation
- Redundant exploration of alternatives during implementation
- Each implementation step includes reasoning from scratch

**With a plan**:
- Implementation phases are pre-scoped and sequenced
- Agent references the saved plan instead of re-explaining
- No need to revisit architectural decisions
- Each step focuses on code, not planning

### Using Plans for Token Efficiency

**When implementing each phase**:
1. **Read the plan file**: `.zed/plans/<session-id>.md` contains all context
2. **Reference specific steps**: Implement Step X.Y from the plan
3. **Delegate with the plan**: Pass the plan file path to delegated agents for context
4. **Minimize re-explanation**: Continue from Phase N in `.zed/plans/<session-id>.md`
5. **Use brief checkpoints**: Completed Phase N per plan. Ready for Phase N+1.

### Token-Efficient Workflow Example

**Initial planning** (full context, comprehensive reasoning):
- User describes task
- Agent uses plan-mode skill, creates detailed multi-phase plan
- Result: Saves `.zed/plans/session-abc123.md`

**Phase 1 implementation** (minimal re-context):
- User: Implement Phase 1 from the plan
- Agent: Reads plan file, implements only Phase 1 steps
- Token saved: No re-explanation of overall architecture

**Phase 2 continuation** (focused, plan-driven):
- User: Next phase
- Agent: References plan for Phase 2, builds on Phase 1
- Token saved: No context re-building, no architectural re-discussion

**Delegating work** (plan as context):
- Agent: Delegates Phase 3 to another agent with plan file path
- Delegated Agent: Reads plan, knows exact scope, no re-planning needed
- Token saved: Delegation does not require re-explaining the full task

## Frugal Planning: High Reasoning, Low Tokens

Achieve deep, strategic reasoning while minimizing token waste. Plan mode supports frugal planning patterns:

### Frugal Planning Principles

**Concentrate reasoning upfront**:
- Spend tokens on planning where they have maximum impact
- Make architectural and design decisions once, not repeatedly during implementation
- Invest in understanding tradeoffs during the planning phase
- Avoid redundant reasoning during implementation

**Compress storage efficiently**:
- Plans should be concise yet complete
- Use structured formats (phases, steps, dependencies)
- Preserve decision rationale without verbose explanation
- Store only information needed for implementation

**Reuse reasoning across phases**:
- Each phase inherits planning decisions from earlier phases
- No need to re-explain or re-reason through architectural choices
- Implementation steps reference the reasoning without repeating it

### Frugal Plan Structure

Create lean plans that preserve full reasoning value:

```markdown
## Approach
[1-2 sentences: core strategy and why]

## Phase 1: [Goal]
- **1.1**: [Action]
  - Rationale: [Why this, not alternatives]
  - Dependencies: [if any]
  - Files: [affected files]
```

Key differences:
- Rationale instead of long explanation (saves words, keeps reasoning)
- List files affected (enables focused implementation)
- Group related steps (reduce overhead)

### Token Usage Pattern with Frugal Planning

**Planning phase** (invest tokens in deep reasoning):
- Full analysis of requirements
- Deep exploration of tradeoffs
- Justification of decisions
- Tokens spent: 2000-5000 (varies by complexity)

**Implementation phases** (minimal tokens per phase):
- Read plan once, implement per plan
- Brief status: "Implementing 1.1-1.3 from plan"
- Ask only about specific blockers, not general context
- Tokens spent: 500-1500 per phase

**Result**: Planning upfront costs 3-5K tokens but saves 2-3K tokens per implementation phase. For a 5-phase project: planning saves 10-15K tokens total.

### Frugal Reasoning Techniques

**Compress decision rationale**:
- Verbose: "We could use Redux (time-travel, ecosystem), Context API (lighter, performance issues), or MobX (intuitive, less popular). We chose Redux because..."
- Frugal: "Redux: debuggability + ecosystem; vs Context API complexity"

**Delegate reasoning to planning**:
- Don't repeat architectural analysis during implementation
- During implementation: "Per plan Phase 2.1, use service layer pattern"
- Implementation focus: execute the decision, don't re-reason it

**Use abbreviations and references**:
- Define once in plan: "Token optimization (TO)"
- Reference thereafter: "Following TO patterns from Phase 1"
- Reduces repetition without losing clarity

**Batch similar decisions**:
- Group related architectural choices in one plan section
- Reduces context switching tokens
- Makes implementation modular without re-explaining context

### Example: Frugal High-Reasoning Plan

**Task**: Build a real-time notification system

**Frugal Plan** (compact, reasoning preserved):

```markdown
## Approach
WebSocket server with Redis pub/sub for horizontal scaling.
Real-time + scalable. Trade-off: more complex than polling.

## Architecture Decisions
- WebSocket vs polling: Lower latency, better resources
- Redis pub/sub vs in-memory: Multiple servers, no session store
- Event-driven state: Simpler than request/response

## Phase 1: WebSocket Infrastructure
- **1.1**: WebSocket server (ws library)
  - Rationale: Lightweight, proven, scales well
  - Files: server/websocket.ts
  
- **1.2**: Redis pub/sub connection
  - Rationale: Horizontal scaling, no session store needed
  - Files: server/redis.ts

## Phase 2: Client Integration
- **2.1**: WebSocket client with auto-reconnect
  - Rationale: Handles network failures transparently
  - Files: client/websocket.ts

## Testing
- Unit: Event handling
- Integration: Publish → client receives
```

**Why this saves tokens while maintaining reasoning**:
- Reasoning compressed but preserved (rationale fields explain "why")
- Architecture decisions stated once (no re-explanation during implementation)
- Clear file targets (implementation stays focused)
- Each phase self-contained (minimal context needed per phase)
- Total: 1500 planning tokens, 400-600 per phase

### When to Spend Tokens (High Reasoning Moments)

**Invest tokens during planning for**:
- Architectural decisions: Right choice prevents costly refactoring
- Integration points: How components talk to each other
- Error handling strategy: How failures are managed
- Performance bottlenecks: Where optimization matters
- Security considerations: Authentication, data access patterns
- Scalability constraints: Production readiness

**Don't spend tokens during implementation on**:
- Re-explaining architecture (reference plan)
- Re-analyzing requirements (reference plan)
- Reconsidering rejected alternatives (plan justifies choice)
- General context rebuilding (read plan file)

### Measuring Frugal Success

- **Planning tokens**: 2-5K (focused deep reasoning)
- **Per-phase tokens**: 400-1000 (focused implementation)
- **Reasoning preserved**: Every key decision has rationale
- **Re-planning needed**: Zero (plan is comprehensive)
- **Total for multi-phase task**: 3-7K tokens (vs 10-20K without planning)

## Implementation After Planning

Once you have created the plan:
1. **Confirm with the user**: Does this approach work for you?
2. **Save the plan**: Plan is automatically saved to `.zed/plans/<session-id>.md`
3. **Start implementation**: Execute each phase as a focused task, referencing the saved plan
4. **Adjust as needed**: If requirements change during implementation, update the plan file
5. **Validate**: Ensure each phase meets its success criteria before moving to the next

The plan file acts as a living document throughout the session, allowing you and the agent to stay aligned on scope and approach while maintaining high reasoning quality and minimizing token overhead.
