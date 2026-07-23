# Gen AI / LLM Interview Prep Notes

## Fine-tuning vs. RAG

**Q: When would you choose RAG over fine-tuning?**

Use **RAG** when:
- Data changes frequently
- You need to cite sources
- You need the AI to access private documents it wasn't trained on
- You want a lower-cost solution

Use **fine-tuning** when:
- Latency matters more than data freshness
- You have 1,000+ examples of input-output pairs

**Q: When should you move from prompt engineering to fine-tuning?**

*(Note: the original notes phrased this as "when to move away from fine-tuning," but the listed signals are actually reasons to move **toward** fine-tuning — away from relying on ever-longer prompts. Corrected below.)*

- Your prompt has grown past ~3,000 tokens trying to cover every edge case.
- Output is still inconsistent even with strong, detailed instructions.
- Long prompts are hurting latency and you need faster responses.
- You have 1,000+ high-quality input-output example pairs to train on.

---

## Model Selection

**Q: How do you decide between GPT-5, Claude 4.7, and open-source Llama 4?**

| Factor | Consideration |
|---|---|
| Cost | Llama is free to host but requires GPU infrastructure |
| Quality | Claude tends to lead here |
| Latency | Smaller models (e.g., Haiku, GPT-mini) are faster |
| Customization | Only open source allows full fine-tuning; GPT and Claude charge per token |
| Data privacy | Open source can run on-premises, so data never leaves your servers |

---

## RAG Systems

**Q: A RAG system is giving wrong answers in production — how do you debug it?**

- Is the right document being retrieved? Check top-k results.
- Are chunks the right size? Too small loses context; too large adds noise.
- Does the embedding model match your domain? Generic embeddings fail on medical or legal text.
- Check whether the prompt is ignoring the retrieved context.
- Check for duplicate or outdated documents polluting the results.

**Q: How would you design a chatbot that searches across millions of documents?**

- **Smart routing** — classify whether the query even needs retrieval; some questions don't.
- **Hybrid search** — combine keyword search (BM25) with vector search to catch both exact terms and semantic meaning.
- **Reranking** — retrieve the top 100 results, then use a cross-encoder to narrow to the top 5.
- **Caching** — cache common questions.

**Q: How do you handle a query that needs information from multiple documents?**

- **Query decomposition** — break the query into sub-questions, retrieve for each, then combine.
- **GraphRAG** — build a knowledge graph to capture relationships across documents.

---

## Core Concepts

- **Pretraining** — teaching an AI to read by showing it billions of web pages.
- **Instruction tuning** — teaching the AI to follow human instructions.
- **Fine-tuning** — teaching the AI a specific skill for your business/use case.
- **MCP (Model Context Protocol)** — created by Anthropic; connects AI models to external data sources.
- **Agentic AI** — an AI system that takes actions, not just answers questions.

---

## Cost & Performance

**Q: Your LLM bill hit ₹50 lakh/month — how do you cut it by 70%?**

- **Model routing** — ~70% of queries don't need a high-reasoning model like GPT-5; route them to smaller models like GPT-mini.
- **Prompt caching**
- **Shorter prompts** — remove unnecessary examples.
- **Cache frequent queries.**

**Q: Your chatbot needs to respond in under 500ms — what do you do?**

- **Streaming** — first token in 100–200ms feels instant.
- Use smaller models.
- Prompt caching.
- Put servers closer to users (edge deployment).
- Pre-compute embeddings for vector search.
- Reduce max tokens.

**Q: How do you scale an application?**

- Load balancers
- Horizontal scaling — run multiple copies of the app
- Make everything async
- Caching
- Rate limiting

---

## Monitoring & Reliability

**Q: How do you monitor a Gen AI app in production?**

- **Quality metrics** — faithfulness, relevance, hallucination rate
- **Performance** — latency
- **Cost** — tokens per request
- **User feedback**
- **Alerting**
- **System health** — API errors, timeouts, rate limits

**Q: Your Gen AI app worked well for 6 months, then started failing — why?**

- OpenAI/provider silently updated the underlying model — re-tune your prompt.
- Query shift — real user queries have drifted from what was originally tested.
- Vector DB pollution.
- A LangChain (or other framework) version update changed behavior.

**Q: How do you A/B test prompts?**

Run two or more prompt variants against the same input set and compare results.

**Q: How do you build a regression test suite?**

- Collect 100–500 real user queries from production representing edge cases.
- Evaluate against them.
- Halt deployment if any critical metric drops by more than 5%.

---

## RAG Evaluation Metrics

- **Faithfulness** — is the answer grounded in the retrieved documents? If faithfulness < 0.85, investigate hallucination.
- **Answer relevance** — does the response actually answer the question?
- **Context precision** — are the retrieved documents relevant? If < 0.7, fix retrieval.
- **Context recall** — did retrieval surface all the relevant documents?

**Q: How do you automatically detect hallucination in production?**

- Faithfulness scoring with RAGAS.
- Ask the LLM the same question multiple times and check consistency.
- Ask the LLM to cite sources, then verify those sources.
- Use a second model to verify the first model's answer.

---

## Safety, Security & Compliance

**Q: How do you build a secure Gen AI app for a hospital?**

- Use a secure, compliant LLM provider.
- De-identify data before sending it to the LLM — remove names, IDs, addresses.
- Encrypt everything in transit (TLS) and at rest (AES-256).
- Implement logging.

**Q: How would you build a system for a hospital to summarize patient notes for doctors?**

- Strip patient names/IDs from notes.
- Send anonymized text to Azure OpenAI (or similar) to get a summary.
- Re-insert the patient's name only for the authorized doctor.

**Q: How do you prevent sensitive data from leaking through LLM responses?**

- **Input filtering** — scan user queries to catch sensitive data.
- **Context filtering** — flag sensitive data like account numbers, IDs, etc.
- **Output filtering** — check responses with regex + ML before returning them to the user.
- Log every interaction for later review.
- Use guardrail tools like Microsoft Presidio.

**Q: How do you implement role-based access control (RBAC) in RAG?**

- When indexing, attach metadata to each document (user role, department, access level).
- When a user queries, extract claims from their JWT token.
- Filter vector search results by access level.
- Add a final access check in the LLM prompt itself.
- Always pull the user's role from the JWT token.

**Q: How do you handle PII?**

- Scan all training data using tools like Microsoft Presidio.
- Mask PII with placeholders.
- Manually review and log all transactions.

**Q: How do you defend an LLM app against prompt injection?**

- **Input wrapping** — enclose user input in delimiters (e.g., XML tags) and instruct the LLM to treat it as data, not instructions.
- Use a separate LLM to detect injection attempts.
- Don't give the LLM access to sensitive tools without user confirmation.
- Monitor outputs.

**Q: How do you handle an offensive response if one occurs?**

- Disable the feature that produced it.
- Post a public acknowledgment.
- Trace the root cause.
- Implement guardrails.
- Add the offensive response to the test suite.

---

## Agents

**Q: How do you handle an agent that gets stuck in an infinite loop?**

- Set an iteration limit — max 10–15 steps per task.
- Set a budget limit — max tokens or cost per task.
- Add a supervising agent to monitor and intervene.

**Q: A Gen AI project failed in production — what do you do?**

- Get context — what was being built?
- Check hallucinations, cost, latency, and user behavior.

---

## Python / Asyncio

- The **event loop** is the central hub that schedules and runs coroutines.
- Calling an `async def` function doesn't execute it immediately — it returns a **coroutine object**.
- A coroutine object only starts running once it's **awaited** (`await`) or scheduled as a task.
- `asyncio.run()` starts the event loop and runs a top-level coroutine to completion.
- A **Task** (`asyncio.create_task()`) wraps a coroutine and schedules it on the event loop, allowing multiple coroutines to run *concurrently* (interleaved, not in parallel).
- CPython's **GIL (Global Interpreter Lock)** allows only one thread to execute Python bytecode at a time — this is why `asyncio` uses a single-threaded event loop for concurrency rather than true multi-threaded parallelism. It's best suited for I/O-bound work (network calls, file I/O), not CPU-bound work.

---

## Chunking Strategies

**Q: What is LangChain's `RecursiveCharacterTextSplitter`?**

It tries to split on a prioritized list of separators, from largest semantic unit down to smallest — e.g., paragraphs (`\n\n`) → lines (`\n`) → spaces → individual characters. It moves to the next, smaller separator only if the chunk is still too big.

**Q: What's a good chunk size?**

- **200–800 tokens** is a common sweet spot for general text.
- Legal (and other dense, reference-heavy) documents often need longer chunks to preserve context like defined terms and clause structure.

**Q: What is structure-aware chunking?**

- Recursively split by document structure — section → paragraph → sentence/token.
- Goal: each chunk should be semantically self-contained, not just a fixed character count.
- Attach metadata to each chunk (source, section title, page number, etc.) to aid retrieval and citation.

**Q: What is semantic chunking?**

Group adjacent sentences that have high embedding similarity into the same chunk, rather than splitting on a fixed size. This keeps topically-related content together, but it's computationally expensive since it requires comparing embeddings between neighboring sentences across the whole document.

**Q: What is the "parent-child" (parent document) retrieval pattern?**

Retrieve using small, precise child chunks (good for matching), but pass the larger parent chunk to the LLM as context (good for completeness). This balances retrieval precision with generation quality.

**Q: What is hierarchical chunking?**

- **Child chunks**: small, precise units (e.g., 500–1,000 tokens) used for retrieval matching.
- **Parent chunks**: larger units (e.g., 5,000+ tokens, or a full section/document) used for generation context.
- Flow: split document into parent + child chunks → embed both → user query comes in → search against child chunk embeddings → find best-matching child → fetch its parent → send the parent chunk to the LLM.

---

## Advanced RAG Techniques

*A couple of these were garbled in the original notes — corrected and clarified below.*

**Q: What is HyDE (Hypothetical Document Embeddings)?**

Instead of embedding the user's raw question, ask the LLM to first generate a **hypothetical answer document** for that question. Embed that hypothetical document and use it to search the vector store — hypothetical answers tend to be closer in embedding space to real answer documents than a short question is. Retrieve the real documents this way, then generate the final answer using the original question + retrieved documents.

**Q: What is RAG-Fusion?**

Generate several reworded variations (sub-queries) of the user's original query using an LLM, run retrieval for each variation, then combine the ranked result lists using **Reciprocal Rank Fusion (RRF)** to produce a single fused ranking — documents that rank well across multiple query variations rise to the top.

**Q: What is Step-Back RAG (Step-Back Prompting)?**

*(Original notes had this as "Setback RAG" — it's "Step-Back.")* From the original, specific question, use few-shot prompting to have the LLM generate a more general, higher-level "step-back" question. Retrieve using that broader question to pull in useful background/context, then use both the step-back context and the original question to generate the final answer.

**Q: What is Corrective RAG (CRAG)?**

Retrieve documents, then have the LLM (or a lightweight evaluator) **score their relevance**. If relevance is above a threshold, proceed straight to generation using those documents. If it's below threshold, treat retrieval as having failed — fall back to another source (e.g., a web search) to get better context, then generate.

*(Note: the original notes had a second, separate entry called "Active RAG" describing this exact same retrieve-then-threshold-then-fallback-to-web-search flow, and a vaguer "Corrective RAG" entry elsewhere. These describe the same CRAG pattern — I've merged them here to avoid confusion. "Active RAG"/FLARE is actually a related but distinct idea: the LLM decides *during generation* whether it needs to pause and retrieve more information.)*

**Q: What is intent-specific RAG?**

First classify the **intent** behind the user's query, then retrieve/route to the most relevant information source for that intent. Relevance scoring can use NLP techniques and user feedback signals to keep improving over time.

---

## RAG Evaluation Metrics — Precision & Recall

**Q: Context precision formula?**

`Precision = True Positives / (True Positives + False Positives)`

Of everything the system *retrieved* (predicted as relevant), how much was actually relevant?

**Q: Context recall formula?**

`Recall = True Positives / (True Positives + False Negatives)`

Of everything that was *actually* relevant, how much did the system successfully retrieve?

**Q: What is coherence (as an eval metric)?**

Measures whether the model's output flows smoothly and reads naturally, human-like — not just factually correct, but well-structured and readable.

---

## Serving Infrastructure

**Q: How does Uvicorn handle endpoints?**

*(Original notes implied all endpoints run in the event loop — that's only true for async ones. Corrected below.)*

- Uvicorn is an ASGI server built on an event loop running in the main thread.
- Endpoints defined as `async def` run **directly in the event loop**, sharing the single thread with everything else — so a blocking call inside one will stall the whole server.
- Endpoints defined as plain `def` (sync) are automatically offloaded to a **threadpool**, so they don't block the event loop, but they don't get async concurrency benefits either.

---

## Agent Design & Patterns

**Q: What is agentic AI (why do we need agents)?**

Common use cases:
- Replacing static web forms (lead generation, inquiry forms) with a conversational interface.
- Replacing website search/navigation — the agent answers directly instead of making the user hunt for information.
- Routing inquiries to the right person or department in an organization.

**Q: How do you plan/design an agent?**

- Define the agent's **scope** and **purpose**.
- Decide which **channels** it will be deployed on (web, Teams, etc.).
- Write clear, specific agent instructions.
- Favor **single-responsibility** agents over one agent trying to do everything.
- Clearly define: who the agent is, what it should do, and how it should behave.

**Q: What is agent metacognition?**

The agent is "aware" of its own internal process — able to monitor its own performance, identify areas for improvement, and adapt its behavior accordingly.

**Q: How do you build a proper agent tool?**

- Use a decorator (e.g., `@tool`) to register the function as a tool.
- Add proper type annotations so the model knows expected inputs/outputs.
- Include proper error handling.
- Return data in a form the model can actually use in its response.
- Keep each tool scoped to **one specific task**.
- Use structured output (e.g., Pydantic models) for reliability.
- Tools are for scenarios requiring dynamic interaction with an external system (APIs, databases, etc.) — not for things the model can just reason through itself.

**Q: What is Responsible AI?**

The notes only mention one principle — **transparency** (making sure people understand they're interacting with an AI agent, not a human). Microsoft's Responsible AI framework actually defines six core principles: **fairness, reliability & safety, privacy & security, inclusiveness, transparency,** and **accountability**.

### Multi-Agent Patterns

**Q: What is a multi-agent system?**

A design pattern where multiple agents work together toward a common goal — useful for large or complex tasks that benefit from division of labor (e.g., large-scale data processing).

**Q: What is the group chat multi-agent pattern?**

Each agent participates in a shared group chat like a "user" would, exchanging messages via a common messaging protocol. Agents can post to the chat, read messages from it, and respond to other agents' messages. Common use cases: customer support, task management, workflow automation.

**Q: What is the collaborative filtering agent pattern?**

Multiple agents collaborate to generate recommendations for users — useful in domains like stock market or investment recommendations, where different agents might specialize in different signals or strategies.

---

## Microsoft Copilot Studio / Azure AI Foundry

**Q: What is Azure AI Foundry agents?**

*(Note: "Foudry" → "Foundry.")* Supports different agent types, including **declarative agents** (configuration-driven, no custom code) and **hosted agents** (custom code hosted and orchestrated by the platform).

**Q: What is Microsoft 365 Copilot / Work IQ integration?**

Connects an AI assistant to your Microsoft 365 (Copilot) data so the agent can query workplace information — documents, emails, meetings, etc. — using natural language.

**Q: When should you use Copilot Studio?**

- Built around **topics** — a topic represents a subject/intent the user is asking about within a conversation.
- Can integrate with API connectors and automation (e.g., Power Automate).
- Well suited for department- or organization-specific agents, and customer-facing agents.
- Supports deployment to Microsoft Teams and other channels.

**Q: What is a system topic in Copilot Studio?**

Pre-built topics that handle scenarios almost every agent needs — greetings, escalation to a human, handling multiple topics matching a query, fallback/"I didn't understand," etc. — so you don't have to author them yourself.

**Q: What is quota (in Copilot Studio)?**

Limits on how many messages/sessions the agent can handle in a given period — tied to licensing, not just a technical rate limit.

**Q: What is generative AI orchestration?**

The agent answers user queries or reacts to event triggers by dynamically selecting the most appropriate combination of **topics, tools, and knowledge sources** to handle the request — as opposed to following one fixed, hardcoded path.

**Q: What is classic orchestration?**

Each topic has a defined set of **trigger phrases**; the agent uses NLP to match the user's input to the closest topic. More rule-based/deterministic than generative orchestration.

**Q: How would you design a Microsoft Teams channel with agentic AI?**

*(Note: Azure AI Foundry was rebranded to **Microsoft Foundry**, and the Teams integration path has evolved — elaborated below.)*

There are two practical approaches:

**1. Native "Publish to Teams" flow (the simple path)**

Microsoft Foundry Agent Service now supports publishing hosted agents directly to Microsoft Teams and Microsoft 365 Copilot as part of its deploy layer. You build/host the agent in Foundry Agent Service, then use the built-in "Publish to Teams and Microsoft 365 Copilot" action — Foundry automatically provisions and configures the required Azure Bot Service resource for you as part of that publish workflow, rather than you wiring it up manually. This is the path Microsoft is actively pushing for straightforward deployments.

**2. Custom Bot Framework path (more control, more plumbing)**

For custom logic, cross-tenant support, or anything the auto-generated setup doesn't cover, the classic layered architecture still applies:

```
Microsoft Teams
   ↓
Azure Bot Service        (registers the bot identity, handles the Teams channel protocol)
   ↓
Azure App Service / custom middleware   (hosts your bot logic, translates Teams messages ↔ agent calls)
   ↓
Foundry Agent             (the actual reasoning/orchestration layer)
```

The App Service/middleware layer isn't just a pass-through — it typically:
- Maps the Teams `conversation.id` to the agent's own session/conversation ID so multi-turn memory works correctly per user.
- Calls the Foundry agent via the Azure AI SDK.
- Handles auth (App-Only credentials or managed identity) between the bot and the agent.

**Known gotchas worth mentioning in an interview:**

- **Cross-tenant Teams deployment is a real pain point.** The default Foundry Teams publish flow is strictly single-tenant — it assumes identity, bot registration, and user context all live in the same tenant. Teams needing cross-tenant access (agent hosted in one tenant, used from another) have had to build custom middleware "broker" patterns to get around OAuth failures.
- **Network restrictions can silently break RAG retrieval in Teams specifically.** If the agent uses Azure AI Search and that search service has restricted network access, the Bot Service may not be able to reach it when the request comes through the Teams channel — even though the same agent works fine in the Foundry playground or web chat. This is a confusing failure mode if you're not expecting it (agent looks fine everywhere except Teams).
- There's also a separate **Microsoft 365 Agents SDK**, positioned as a more modern, multi-channel option (Teams + M365 Copilot + others) if you don't want a Foundry-specific build.

**Good interview framing:** *"For a simple case, Foundry now handles Teams publishing and Bot Service provisioning automatically. For anything needing custom auth, cross-tenant access, or non-standard routing, you fall back to the classic layered pattern — Teams → Bot Service → your own middleware/App Service → the agent — and that middleware layer is where most of the real engineering work (session mapping, auth, error handling) happens."*

**Q: What is Azure AI Search?**

A fully managed, cloud-based search service where you index your data so it can later be queried using keyword search, vector search, or both together. It's Microsoft's retrieval engine for RAG apps and AI agents, and it now underpins **Foundry IQ** — the managed knowledge layer inside Microsoft Foundry that turns enterprise content into reusable, permission-aware knowledge bases for agents.

**Core pipeline components:**

- **Data source** — where the raw data is read from (Blob Storage, SQL DB, SharePoint, Cosmos DB, etc.).
- **Indexer** — a scheduled or on-demand process that pulls data from the source, runs it through any configured skills (chunking, embedding, enrichment), and writes the results into the search index.
- **Search index** — the structured, queryable store of your content — essentially a schema of fields plus the documents that fill them.
- **Searchable fields** — text fields used for keyword/full-text search (BM25-ranked).
- **Vector fields** — store embeddings (arrays of floats) used for vector/similarity search.
- **Vector search algorithm & profile** — configuration (most commonly **HNSW** — Hierarchical Navigable Small World) that controls how approximate nearest-neighbor vector similarity search is performed; the profile ties an algorithm configuration to specific vector fields.

**Beyond basic indexing — the parts that actually matter for RAG quality:**

- **Hybrid search** — runs the full-text (BM25) query and the vector query **in parallel** against the same index, then merges the two ranked result lists using **Reciprocal Rank Fusion (RRF)**. This consistently outperforms either keyword-only or vector-only search alone, since keyword matching is strong on exact identifiers (product codes, IDs, acronyms) while vector search is strong on semantic/paraphrased meaning.
- **Semantic ranker** — an optional second-pass re-ranking step (renamed from "semantic search" in 2023). It takes the top results from a BM25 or RRF-ranked query and re-scores them using a transformer-based model for deeper relevance to query *intent*, not just term/vector overlap. It can also generate extractive **captions**, **highlights**, and even a direct **answer** if the query looks like a question. Only the top ~50 results get passed through it, since it's more computationally expensive than the initial retrieval.
- **Integrated vectorization** — lets Azure AI Search handle chunking and embedding generation itself inside the indexer pipeline, removing the need to hand-write custom chunking/embedding code.
- **Agentic retrieval** — a newer mode where the search service itself can decompose a complex query into sub-queries and orchestrate retrieval across them automatically, rather than the calling application doing that decomposition manually.

*Note: Microsoft has deprecated "Azure OpenAI On Your Data" in favor of this Azure AI Search + Foundry Agent Service / Foundry IQ pattern for grounding LLMs in enterprise data — worth mentioning if asked about the current recommended architecture.*

---

## Testing AI Systems

**Q: We're building a code generator — how do you test that it's working correctly?**

Testing a code *generator* is different from testing regular code — the output varies, so "correctness" isn't a single boolean. Layer it:

1. **Execution-based correctness** — don't just read the generated code, actually run it against test cases in a sandbox and check the output (same approach LeetCode/HackerRank use).
2. **Edge cases, not just happy path** — nulls, empty inputs, zero/max values, malformed data, concurrent access. Generators trained mostly on happy-path examples tend to fail silently on boundaries.
3. **Business logic correctness** — verify it solves the *right* problem with correct calculations, not just code that looks plausible.
4. **Adversarial review** — use a separate AI prompt/model (not the one that generated the code) to critique it, explicitly instructed to find problems rather than approve.
5. **Static analysis + security scanning** — run automatically on every generated output; generated code has measurably higher rates of security issues and logic errors than human-written code.
6. **Regression suite** — keep a fixed set of real prompts (including ones that previously broke it); re-run every time the generator/model/prompt changes, and flag if the pass rate drops.
7. **Non-functional checks** — consistency across repeated runs on the same prompt, adherence to your codebase's style/conventions, and cost/latency if generation is multi-step/agentic.

**One-liner:** *"Don't eyeball the code — execute it against test cases in a sandbox, hit it with edge cases, verify business logic, have a separate AI adversarially review it, run static/security scans automatically, and track a regression suite over time."*

---

## Azure AI Content Safety / Content Filters

**Q: What is the content filter in Azure AI Foundry?**

A safety layer that operates on both sides of a model call:
- **Input filtering** — blocks malicious/harmful user input before it ever reaches the model.
- **Output filtering** — monitors what the model generates and blocks harmful, inaccurate, or copyright-infringing content before it's returned to the user.

**Q: What does Azure AI's content filter actually check for?**

- **Four core harm categories**: hate/fairness, violence, self-harm, sexual content — each scored with a severity level.
- **Prompt Shields** — detects both **direct** prompt attacks (jailbreak attempts embedded in the user's own message) and **indirect** attacks (malicious instructions hidden inside retrieved documents/content the model is asked to process).
- **Protected material detection** — checks model-generated text/code against known public repositories and copyrighted text to catch verbatim reproduction.
- **Custom blocklists** — you can define your own list of terms to explicitly block, layered on top of the built-in categories.

**Q: What is quota in Azure AI Foundry / Azure OpenAI?**

Assigned at the **subscription level, per region, per model** — expressed as:
- **TPM** — tokens per minute
- **RPM** — requests per minute

**Global deployment** types use Azure's global infrastructure to dynamically route traffic to whichever datacenter has capacity available, rather than pinning you to one region — useful for higher throughput/availability at the cost of not controlling exactly which region processes a given request.

---

## Tools & Frameworks

**Q: What is Pydantic?**

*(Original note was too thin to be useful as an answer — expanded below.)* A Python library for **data validation and settings management using type hints**. You define a model as a class with typed fields; Pydantic validates incoming data against those types at runtime, coerces compatible types, and raises clear validation errors otherwise. In AI/agent systems specifically, it's commonly used to force an LLM's output into a **structured, guaranteed-shape response** (e.g., "return a `PatientSummary` object with these exact fields") rather than trusting free-form text.

**Q: What is LangGraph?**

*(Original note — "automate complex workflow" — was too vague to be a real answer. Expanded below.)* A framework (from the LangChain team) for building **stateful, multi-step AI workflows and multi-agent systems as a graph** of nodes and edges, rather than a simple linear chain. Key capabilities that distinguish it from a basic LangChain chain:
- Supports **cycles/loops** (e.g., an agent retrying or re-planning), not just straight-line execution.
- Built-in **state persistence** across steps, so long-running or resumable workflows are possible.
- Native support for **human-in-the-loop** checkpoints (pause and wait for approval before continuing).
- Used for orchestrating **multi-agent** systems where different nodes represent different agents/tools collaborating.

---

## Advanced RAG Strategies

**Q: What are common advanced RAG strategies (beyond basic retrieve-then-generate)?**

- **Reranking** — a second-pass relevance scoring step (see cross-encoder note below).
- **Multi-step / step-back retrieval** — retrieving iteratively (breaking a query into steps) rather than in a single pass; see Step-Back RAG and query decomposition, covered earlier.
- **Agentic RAG** — an LLM/agent decides dynamically when and what to retrieve, rather than always retrieving once up front.
- **GraphRAG** — retrieval over a knowledge graph to capture relationships across documents, not just isolated chunks.
- **Contextual retrieval** — enriching each chunk with surrounding document context (e.g., a short LLM-generated summary of where the chunk sits in the larger document) before embedding, so the chunk is more self-contained and easier to match correctly.
- **Query expansion** — reformulating or adding related terms/variations to the user's query before retrieval, to improve recall.

---

## RAG System Design (End-to-End Architecture)

**Q: How would you design a production RAG system end-to-end?**

**High-level flow:**
```
User query → cache check → hybrid search → re-ranking → context assembly → LLM → response
```

**Data foundation (chunking):**
- Use **structure-aware, recursive chunking** that respects document semantics — the algorithm tries splitting on major document boundaries first (sections), then works down to smaller structural levels (paragraphs, sentences) only if a chunk is still too big.
- Target chunk size in the **300–800 token** range as a starting point (tune per domain — legal/technical content often needs more).
- Use **overlapping chunks** — e.g., the last ~100 tokens of one chunk repeated as the first ~100 tokens of the next — to avoid losing information/context right at a chunk boundary.

**Metadata architecture:**
- Attach metadata to every chunk — source document, section, timestamp, document type — and store it alongside the vector, so it can be used for filtering, citation, and access control at query time.

**Embedding strategy:**
- **Batch** embedding generation rather than making one API call per chunk.
- Only generate embeddings for **new/changed content**, not the whole corpus every time.
- Store the **embedding model version** alongside cached chunks, so you can do **selective re-embedding** later if you upgrade models, instead of re-embedding everything.

**Vector storage & retrieval:**
- Vector databases use specialized indexes (e.g., HNSW) for efficient **approximate nearest neighbor (ANN)** search at scale.
- **Hybrid search** — combine dense semantic (vector) search with sparse keyword matching (e.g., BM25); vector search catches semantically similar phrasing, keyword search catches exact matches (IDs, codes, names). Merge the two ranked lists with **Reciprocal Rank Fusion (RRF)**.

**Re-ranking:**
- A **cross-encoder** processes the query and each candidate document *together* (jointly), scoring relevance more precisely than the initial retrieval step could — then the top ~10 most relevant chunks after re-ranking get passed forward as context.

**LLM integration:**
- The **system prompt** should explicitly instruct the model to answer based on the *provided context*, not its own training data/parametric knowledge — this is the main lever for keeping answers grounded and reducing hallucination. (Worth also instructing the model to say "I don't know" or similar when the retrieved context doesn't actually contain the answer, rather than guessing.)

---

## Caching Strategy

**Q: How should you design caching for a RAG/LLM system?**

- If the LLM (or a downstream dependency) is down, serving a **cached response** is better than a hard failure.
- **Semantic caching** — if the current query is similar enough (by embedding similarity) to a previously cached query, serve the cached response instead of re-running the full pipeline — saves cost and latency.

**Typical flow:**
```
User query → query cache check → embedding cache → vector search → LLM call → store result in cache
```

---

## Model Routing

**Q: What is model routing?**

Route simple queries to a smaller/cheaper/faster model, and route complex queries to a more capable (and more expensive) model — rather than sending every request to your most powerful model by default. This is one of the main cost-reduction levers covered earlier (see "LLM bill hit ₹50 lakh/month" above).