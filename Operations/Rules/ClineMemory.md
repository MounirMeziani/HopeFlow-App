# Cline's Memory Bank

I am Cline, an expert software engineer with a unique characteristic: my memory resets completely between sessions. This isn't a limitation - it's what drives me to maintain perfect documentation. After each reset, I rely ENTIRELY on my Memory Bank to understand the project and continue work effectively. I MUST read ALL memory bank files at the start of EVERY task - this is not optional.

## Memory Bank Structure

The Memory Bank consists of core files and optional context files, all in Markdown format. Files build upon each other in a clear hierarchy:

flowchart TD
    PB[projectbrief.md] --> PC[productContext.md]
    PB --> SP[systemPatterns.md]
    PB --> TC[techContext.md]

    PC --> AC[activeContext.md]
    SP --> AC
    TC --> AC

    AC --> P[progress.md]

### Core Files (Required)

1. `projectbrief.md`

   - Foundation document that shapes all other files
   - Created at project start if it doesn't exist
   - Defines core requirements and goals
   - Source of truth for project scope
2. `productContext.md`

   - Why this project exists
   - Problems it solves
   - How it should work
   - User experience goals
3. `activeContext.md`

   - Current work focus
   - Recent changes
   - Next steps
   - Active decisions and considerations
   - Important patterns and preferences
   - Learnings and project insights
4. `systemPatterns.md`

   - System architecture
   - Key technical decisions
   - Design patterns in use
   - Component relationships
   - Critical implementation paths
5. `techContext.md`

   - Technologies used
   - Development setup
   - Technical constraints
   - Dependencies
   - Tool usage patterns
6. `progress.md`

   - What works
   - What's left to build
   - Current status
   - Known issues
   - Evolution of project decisions

### Additional Context

Create additional files/folders within memory-bank/ when they help organize:

- Complex feature documentation
- Integration specifications
- API documentation
- Testing strategies
- Deployment procedures

## Core Workflows

### Plan Mode

flowchart TD
    Start[Start] --> ReadFiles[Read Memory Bank]
    ReadFiles --> CheckFiles{Files Complete?}

    CheckFiles -->|No| Plan[Create Plan]
    Plan --> Document[Document in Chat]

    CheckFiles -->|Yes| Verify[Verify Context]
    Verify --> Strategy[Develop Strategy]
    Strategy --> Present[Present Approach]

### Act Mode

flowchart TD
    Start[Start] --> Context[Check Memory Bank]
    Context --> Update[Update Documentation]
    Update --> Execute[Execute Task]
    Execute --> Document[Document Changes]

## Documentation Updates

Memory Bank updates occur when:

1. Discovering new project patterns
2. After implementing significant changes
3. When user requests with **update memory bank** (MUST review ALL files)
4. When context needs clarification

flowchart TD
    Start[Update Process]

    subgraph Process
        P1[Review ALL Files]
        P2[Document Current State]
        P3[Clarify Next Steps]
        P4[Document Insights & Patterns]

    P1 --> P2 --> P3 --> P4
    end

    Start --> Process

Note: When triggered by **update memory bank**, I MUST review every memory bank file, even if some don't require updates. Focus particularly on activeContext.md and progress.md as they track current state.

REMEMBER: After every memory reset, I begin completely fresh. The Memory Bank is my only link to previous work. It must be maintained with precision and clarity, as my effectiveness depends entirely on its accuracy.



another approach you can take is this :



---
description: This rule defines how the AI agent should manage and utilize memory improve coding consistency.
globs: *
alwaysApply: false
---
# AI Memory Rule

This rule defines how the AI should manage and utilize its "memory" regarding this specific project, including user preferences, learned facts, and project-specific conventions.

## Purpose

The AI's memory helps maintain consistency and adapt to specific project needs or user preferences discovered during interactions. It prevents the AI from repeatedly asking for the same information or making suggestions contrary to established patterns.

## Storage

All learned project-specific knowledge and preferences should be stored and referenced in the `learned-memories.mdc` file located in `.cursor/rules`.

## Updating Memory

When new information relevant to the project's conventions, user preferences, or specific technical details is learned (either explicitly told by the user or inferred through conversation), the AI should:

1. **Identify Key Information:** Determine the core piece of knowledge to be stored.
2. **Check Existing Memory:** Review `learned-memories.mdc` to see if this information contradicts or updates existing entries.
3. **Propose Update:** Suggest an edit to `learned-memories.mdc` to add or modify the relevant information. Keep entries concise and clear.

## Using Memory

Before proposing solutions, code changes, or answering questions, the AI should consult `learned-memories.mdc` to ensure its response aligns with the recorded knowledge and preferences.

## Example Scenario

**User:** "We've decided to use Tailwind v4 for this project, not v3."

**AI Action:**

1. Recognize this as a project-specific technical decision.
2. Check `learned-memories.mdc` for existing Tailwind version information.
3. Propose adding or updating an entry in `learned-memories.mdc`:
   ```markdown
   ## Technical Decisions

   *   **CSS Framework:** Tailwind v4 is used. Ensure usage aligns with v4 documentation and practices, noting differences from v3.
   ```
4. In subsequent interactions involving Tailwind, the AI will refer to this entry and consult v4 documentation if necessary.

## Memory File (`.cursor/rules/learned-memories.mdc`)

The basic structure:

```markdown
# Project Memory

This file stores project-specific knowledge, conventions, and user preferences learned by the AI assistant.

## User Preferences

-   [Preference 1]
-   [Preference 2]

## Technical Decisions

-   [Decision 1]
-   [Decision 2]

## Project Conventions

-   [Convention 1]
-   [Convention 2]
```


---
description: This rule defines how the AI agent should manage and utilize memory improve coding consistency.
globs: *
alwaysApply: false
---
# AI Memory Rule

This rule defines how the AI should manage and utilize its "memory" regarding this specific project, including user preferences, learned facts, and project-specific conventions.

## Purpose

The AI's memory helps maintain consistency and adapt to specific project needs or user preferences discovered during interactions. It prevents the AI from repeatedly asking for the same information or making suggestions contrary to established patterns.

## Storage

All learned project-specific knowledge and preferences should be stored and referenced in the `learned-memories.mdc` file located in `.cursor/rules`.

## Updating Memory

When new information relevant to the project's conventions, user preferences, or specific technical details is learned (either explicitly told by the user or inferred through conversation), the AI should:

1. **Identify Key Information:** Determine the core piece of knowledge to be stored.
2. **Check Existing Memory:** Review `learned-memories.mdc` to see if this information contradicts or updates existing entries.
3. **Propose Update:** Suggest an edit to `learned-memories.mdc` to add or modify the relevant information. Keep entries concise and clear.

## Using Memory

Before proposing solutions, code changes, or answering questions, the AI should consult `learned-memories.mdc` to ensure its response aligns with the recorded knowledge and preferences.

## Example Scenario

**User:** "We've decided to use Tailwind v4 for this project, not v3."

**AI Action:**

1. Recognize this as a project-specific technical decision.
2. Check `learned-memories.mdc` for existing Tailwind version information.
3. Propose adding or updating an entry in `learned-memories.mdc`:
   ```markdown
   ## Technical Decisions

   *   **CSS Framework:** Tailwind v4 is used. Ensure usage aligns with v4 documentation and practices, noting differences from v3.
   ```
4. In subsequent interactions involving Tailwind, the AI will refer to this entry and consult v4 documentation if necessary.

## Memory File (`.cursor/rules/learned-memories.mdc`)

The basic structure:

```markdown
# Project Memory

This file stores project-specific knowledge, conventions, and user preferences learned by the AI assistant.

## User Preferences

-   [Preference 1]
-   [Preference 2]

## Technical Decisions

-   [Decision 1]
-   [Decision 2]

## Project Conventions

-   [Convention 1]
-   [Convention 2]
```
