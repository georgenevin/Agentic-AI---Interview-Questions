# Agent Skills

## 1. Skill Folder Structure

An agent skill should be organized inside its own folder:

```text
skill/
├── skill.md
├── scripts/
├── references/
└── assets/
```

### `skill.md`

`skill.md` is the main instruction file for the skill.

Keep it **below 500 lines**. The goal is to keep the core instructions focused and easy for the model to process.

A shorter, clearly written skill is generally easier to maintain and less likely to introduce conflicting or ambiguous instructions.

### Supporting folders

Use the supporting folders for information that does not need to be present in the main skill instructions:

* **`scripts/`** — executable scripts or utilities used by the skill.
* **`references/`** — detailed documentation, guides, schemas, examples, and other reference material.
* **`assets/`** — supporting files required by the skill, such as templates or other static resources.

Avoid putting large amounts of documentation directly into `skill.md`. Move detailed or multi-page material into reference files and instruct the agent when those files should be consulted.

---

# 2. Two Types of Skills

There are two primary ways a skill can be invoked:

1. **Model-invoked skills**
2. **User-invoked skills**

## 2.1 Model-Invoked Skills

A model-invoked skill is triggered **autonomously by the agent**.

The agent decides whether the skill is relevant based primarily on the skill's frontmatter, especially its `description`.

Use this type when:

> The agent should decide on its own when the skill needs to be used.

For example, if a skill provides capabilities for analysing financial reports, the agent should recognize when the user's request requires that capability and invoke the skill automatically.

### Key consideration

The description needs to tell the model both:

* **What** the skill can do.
* **When** it should be triggered.

A description that only explains the capability may not provide enough information for reliable invocation.

---

## 2.2 User-Invoked Skills

A user-invoked skill is triggered explicitly by the user, typically through a named or slash-command invocation.

For example:

```text
/my-skill
```

Use this type when:

> The workflow should only execute when the user intentionally asks for it.

This is useful for deliberate workflows where automatic invocation would be undesirable.

Examples include:

* `/generate-report`
* `/review-document`
* `/deploy`
* `/create-presentation`

The important difference is **who decides to invoke the skill**:

| Skill type    | Invocation decision |
| ------------- | ------------------- |
| Model-invoked | Agent decides       |
| User-invoked  | User decides        |

---

# 3. Skill Description: Define the "What" and the "When"

For model-invoked skills, the description is especially important.

A good description should contain two pieces of information:

### What

Describe the capability the skill provides.

> "Analyses financial reports and identifies revenue, cost, and profitability trends."

### When

Describe the context in which the agent should use it.

> "Use when the user asks to analyse financial reports, compare financial performance, or identify profitability trends."

Combined:

> "Analyses financial reports and identifies revenue, cost, and profitability trends. Use when the user asks to analyse financial reports, compare financial performance, or identify profitability trends."

This gives the model both the **capability** and the **trigger context**.

### General pattern

```text
[What the skill does].
Use when [specific context in which the skill should be triggered].
```

Avoid vague descriptions such as:

```text
A skill for working with documents.
```

Prefer:

```text
Extracts structured information from business documents and summarizes key findings. Use when the user asks to analyse, extract information from, or summarize business documents.
```

---

# 4. Frontmatter

The frontmatter normally contains the skill's basic metadata, including:

```yaml
---
name: financial-analysis
description: Analyses financial reports and identifies revenue, cost, and profitability trends. Use when the user asks to analyse financial reports or compare financial performance.
---
```

The **name** identifies the skill.

The **description** explains its capability and invocation context.

For model-invoked skills, the frontmatter description is particularly important because it is available to the model as part of the context used to determine whether the skill is relevant.

Therefore:

> Keep the frontmatter description minimal, precise, and highly discriminative.

Do not turn the description into the complete skill instructions.

---

# 5. Keep `skill.md` Focused

`skill.md` should contain the instructions necessary for the agent to use the skill correctly.

It should not become a large documentation repository.

Instead of putting a large guide directly into:

```text
skill.md
```

move it into:

```text
references/
```

For example:

```text
financial-analysis/
├── skill.md
├── scripts/
│   └── calculate_metrics.py
├── references/
│   ├── financial-metrics.md
│   ├── reporting-guide.md
│   └── examples.md
└── assets/
    └── report-template.xlsx
```

The skill can then tell the agent when to consult those references.

This keeps the main skill instructions smaller and makes the supporting knowledge easier to maintain.

---

# 6. Describe the Desired Outcome, Not a Rigid Procedure

A common mistake when writing skills is to describe an overly rigid sequence of steps.

For example:

```text
First do A.
Then do B.
Then do C.
Then call tool X.
Then perform D.
Finally produce E.
```

This can unnecessarily constrain the agent.

Instead, describe the **desired outcome and important constraints**.

For example:

```text
Analyse the report and produce a summary containing:
- key financial metrics
- significant changes
- major risks
- actionable observations

Use the available financial data and supporting references when necessary.
```

The second approach gives the agent room to determine the appropriate execution strategy while still defining the expected result.

### General principle

> Define **what must be achieved**, rather than unnecessarily prescribing **exactly how it must be achieved**.

---

# 7. Constraints vs. Procedures

Use **constraints** when they matter to correctness.

For example:

```text
- Do not invent missing financial values.
- Clearly distinguish calculated values from values provided in the source.
- Cite the source when reporting information from the reference documents.
```

These are useful because they define boundaries.

Avoid unnecessary procedural instructions such as:

```text
Step 1: Open file A.
Step 2: Read lines 1–50.
Step 3: Copy the values.
Step 4: Calculate X.
Step 5: Call tool Y.
```

Unless the exact procedure is required for correctness, the agent should generally have flexibility in determining how to achieve the outcome.

---

# 8. Define When the Skill Should NOT Fire

For model-invoked skills, describing when **not** to use the skill can be as important as describing when to use it.

This helps prevent overlapping skills from being triggered for unrelated requests.

For example:

```text
Use this skill when the user requests financial analysis.

Do not use this skill for:
- general accounting questions
- simple arithmetic
- requests unrelated to financial reports
- generating fictional financial data
```

The exclusion criteria should be specific and useful rather than excessive.

### Why this matters

Imagine two skills:

```text
Skill A: Analyse financial reports.
Skill B: Answer general accounting questions.
```

Without clear boundaries, the agent may incorrectly select Skill A for a basic accounting question.

A good skill description should make the distinction clear.

---

# 9. Practical Skill-Writing Principles

When creating a skill, use the following checklist:

### Structure

* Keep `skill.md` below 500 lines.
* Keep the main skill instructions focused.
* Put detailed documentation in `references/`.
* Put reusable execution logic in `scripts/`.
* Put supporting templates/files in `assets/`.

### Description

For model-invoked skills:

* Clearly describe **what** the skill does.
* Clearly describe **when** it should be triggered.
* Keep the description concise.
* Avoid vague descriptions.
* Define important exclusion cases where necessary.

### Instructions

* Focus on the desired outcome.
* State important constraints.
* Avoid unnecessary rigid procedures.
* Give the agent reasonable flexibility in execution.
* Avoid duplicating information that belongs in reference files.

### Invocation

* Use **model invocation** when the agent should autonomously determine when the capability is needed.
* Use **user invocation** when the user should intentionally start the workflow.

---

# 10. Core Mental Model

A useful way to think about a skill is:

```text
                 SKILL
                   │
        ┌──────────┴──────────┐
        │                     │
   CAPABILITY              TRIGGER
     "What"                 "When"
        │                     │
 What can the skill       When should the
 accomplish?              skill be used?
        │                     │
        └──────────┬──────────┘
                   │
             INSTRUCTIONS
                   │
          What outcome and
           constraints are
              required?
                   │
        ┌──────────┼──────────┐
        │          │          │
     Scripts   References   Assets
```

The central principle is:

> **A good skill tells the agent what capability it provides, when that capability is relevant, what outcome is expected, and what constraints must be respected—without unnecessarily forcing a rigid execution path.**

### LangGraph

LangGraph does **not automatically read `SKILL.md`** from the repository.

If the runtime agent needs the skill:

```text
SKILL.md
   ↓
Your code loads it
   ↓
Agent instructions/context
   ↓
LangGraph Agent
```

---

## Development vs Runtime

Keep these separate:

```text
Development:
Developer → Copilot → Skill/project instructions → Code

Runtime:
User → LangGraph Agent → Instructions + Knowledge + Tools → Result
```

A skill used by Copilot during development does **not automatically become context for your runtime agent**.

---

## Writing Skills

Prefer:

**Desired Outcome + Constraints**

over:

**Rigid Step-by-Step Procedure**

```text
Validate passenger information and generate a valid
boarding pass.

Do not generate one when mandatory information is missing.
```

Give the agent flexibility unless a specific procedure is required for correctness.

---

# Minimal `SKILL.md` Template

```markdown
---
name: <skill-name>
description: <What the skill does>. Use when <when it should be triggered>.
---

# <Skill Name>

## Purpose

<Briefly describe the capability and desired outcome.>

## Instructions

- <Important instruction>
- <Important constraint>
- <Expected behavior/output>

## Do Not Use When

- <Important exclusion, if applicable>

## References

- Consult `references/<file>.md` when <condition>.
```

### Example

```markdown
---
name: boarding-pass
description: Generates and validates boarding passes. Use when the user requests boarding-pass generation or validation.
---

# Boarding Pass

## Purpose

Generate a valid boarding pass using passenger and flight information.

## Instructions

- Validate mandatory passenger and flight details.
- Follow the business rules defined in the BRD.
- Do not generate a boarding pass when required information is missing.

## References

- Consult `references/boarding-pass-brd.md` for business rules.
```

---

## Quick Rules

* `SKILL.md` → **instructions for a reusable capability**
* `references/` → **supporting knowledge**
* `scripts/` → **executable utilities**
* `assets/` → **supporting files/templates**
* BRD → **business requirements**
* Model-invoked → **agent decides**
* User-invoked → **user decides**
* LangGraph → **must explicitly load skill information**
* Keep **instructions, knowledge, tools, and orchestration** separate
* Define **what + when + important constraints**

Agent Skills — Key Points

1. Progressive Disclosure

Skills should provide information in layers:

Skill Description
       ↓
SKILL.md
       ↓
references/

Keep the skill description small and focused.

SKILL.md contains the main instructions and details.

Put deep, detailed information in references/.

The agent should go deeper into references only when needed.

2. Capability vs Preference Skills

Capability Skills

Teach the model something it cannot currently do reliably.

Examples:

Tracing logs

Creating a React application

These skills may become unnecessary as models improve.

Preference Skills

Capture organization- or team-specific preferences.

Examples:

Company-specific workflows

Coding conventions

Writing/style preferences

These are generally more durable because they represent requirements specific to the organization or use case.

3. Skill vs Deterministic Workflow

Do not use a skill when the workflow is always the same.

If a process is deterministic:

Fixed workflow → Script / Tool

Use a skill when the agent needs flexibility to determine how to achieve a goal.

Principle:

Define goals and constraints rather than forcing a rigid step-by-step procedure.

4. Evaluate Skills

Skills should be tested before being used in production.

Start with:

Happy-path test cases

Negative test cases

Real user/production examples when available

A simple starting point is:

5 positive cases
+
5 negative cases

Negative tests are important to verify that the skill does not trigger when it should not.

5. Test Outcomes, Not Execution Paths

Evaluate whether the agent successfully achieves the desired outcome.

Do not require the skill to be triggered at a specific turn or follow a specific execution path.

Desired outcome
      ↓
   Success?

The agent may achieve the correct result without using the skill, and that can be acceptable.

6. Regression Testing

Agents are nondeterministic, so a single successful run is not enough.

Run multiple trials for each test case.

A practical range is around 3–6 trials per case.

Re-run evaluations whenever the skill changes.

Use isolated environments when possible to avoid previous context affecting results.

7. Retire Skills When They Are No Longer Needed

Skills do not need to exist forever.

Run evaluations:

With Skill
    vs
Without Skill

If the model performs equally well without the skill, consider retiring it.

Keep the evaluations even after removing the skill. They can detect future model-performance degradation and help determine whether the skill needs to be reintroduced.

8. Development-Time vs Runtime Agents

Keep these concepts separate:

Development
Developer → Copilot/Coding Agent → Skills → Build the application

Runtime
User → Application Agent → Instructions + Knowledge + Tools → Result

A skill available to a coding agent does not automatically become context for a runtime agent such as a LangGraph agent.

If the runtime agent needs the skill, the application must explicitly provide/load the relevant skill information.

