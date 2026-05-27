# Prior art

Two voices in the field have converged on the same shape this skill produces. Cite them when the user asks "where does this pattern come from?"

## Addy Osmani — *How to write a good spec for AI agents*

https://addyosmani.com/blog/good-spec/

### Core ideas

- **Specs are PRDs for literal-minded agents.** Be specific about inputs, outputs, constraints. "Most agent files fail because they're too vague."
- **Six required sections** in his canonical form: commands (with full flags), testing approach, project structure, code style (with examples), git workflow, and **boundaries**.
- **Three-tier constraint system** — always do / ask first / never do — embedded in every spec.
- **Hierarchical formatting.** Markdown headings, clear sections, parseable structure.
- **Bring domain expertise into specs** to steer agents away from common pitfalls.
- **Modular over monolithic.** Break large tasks into focused prompts rather than dumping 50 pages of docs and hoping the model extracts what's relevant.
- **Don't confuse rapid prototyping with production engineering.** Specs for vibes ≠ specs for shipping.

### What we adopted

- Boundaries → our **Out of scope** section (mandatory, never empty).
- Domain expertise → our **Implementation notes** section (gotchas + patterns + non-obvious whys).
- Modular → 3–12 children per epic, not one mega-issue.

## Tom Yedwab — *Technical Design Spec pattern* (referenced by Arguing with Algorithms)

https://www.arguingwithalgorithms.com/posts/technical-design-spec-pattern.html

### Core ideas

- **Force the agent to commit plans to writing before code.** Prevents the "death spiral" of cascading mid-stream errors.
- **Spec as long-term memory.** Keeps the agent focused; the spec survives context-window flushes.
- **One step at a time + checkpoint after each.** Don't try a big bang implementation.
- **Full file paths everywhere.** Prevents the classic "agent recreated the file in the wrong directory" bug.
- **When the agent fails, revert + update the prompt — don't ask for corrections.** Saves context budget.
- **Strong-reasoning model for spec creation, faster model for execution.** Split the work.

### Typical sections in Yedwab's spec

- Overview of feature goals
- Database models with full schema definitions
- Event definitions with payload structures
- State handlers describing data mutation logic
- API specifications (endpoints, parameters, responses, error codes)
- Data validation requirements
- Registration instructions for system integration
- **Ordered task breakdown** for sequential implementation
- Testing and documentation expectations

### What we adopted

- "Full file paths everywhere" → our **What exists today** section.
- Ordered task breakdown → our **Acceptance criteria** sub-grouped checkbox list.
- Spec as long-term memory → issue body is the durable memory; the agent reads only the issue, not the conversation that produced it.
- Schema definitions inline → reflected in **Implementation notes** and **What exists today**.

## Where we differ

- Yedwab does one mega-spec per feature. We do one **epic + N children** because GitHub issues are the unit of agent assignment.
- Osmani's three-tier constraint (always / ask first / never) is implicit in our pattern rather than a separate section — usually inferred from "Acceptance criteria" (do this) + "Out of scope" (don't do this) + the project's own CLAUDE.md (always check this).
- We bias harder toward **persona naming.** "Dr Patel" beats "the user." Concrete personas catch product-level confusion before code-level confusion.

## How to cite

When the user asks "where does this template come from?", give the two URLs above and one-line summaries. Don't lecture — the field is converging and we're following.
