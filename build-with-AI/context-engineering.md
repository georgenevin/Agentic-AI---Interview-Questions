Step 1: Ask the AI to Analyze the Codebase

Analyze this repository like a senior engineer who is onboarding to the team.
Generate a CONTEXT.md file that another AI agent could use to understand and work on this project without reading the entire codebase.
- Project purpose
- Architecture
- Main components
- Data flow
- Business rules you can infer
- Coding patterns
- Important dependencies
- Key entry points
- Build/deployment process
- Known risks and technical debt
Do not make assumptions.
Mark any uncertain information.

Step 2: Manually Review the Generated Context

Step 3: Save It in the Repository

repo/
├── CONTEXT.md
├── README.md
├── src/
└── tests/

or  

repo/
├── docs/
│   └── CONTEXT.md
└── src/
``

Step 4: Update It Over Time

Update CONTEXT.md to reflect:
- New services
- New business rules
- Architecture changes
- Important design decisions

Step 5: Start Every New AI Session With It

Read CONTEXT.md first.

Summarize your understanding of the project.

Then help me implement feature X.

Step 6: Don't Let Context Grow Forever
instead of entering every details in one chat
Current chat
→ summarize findings
→ new chat
→ load summary

Step 7 : Make AI explain before acting

Explain:

- What the problem is
- Which files are involved
- Why those files matter
- What changes are required

Do not write code yet.

Takeaway 
The best prompt is often:
What information do you need before you can confidently implement this feature?
Let the AI ask questions and identify missing context before coding.

This is how senior engineers work in real projects. They don't start coding immediately. They first gather context, build a mental model, then implement.

In one sentence: Dex's real message is not "create a CONTEXT.md file." It's "treat context as a first-class artifact and spend as much effort managing context as you spend writing code."

