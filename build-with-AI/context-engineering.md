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
