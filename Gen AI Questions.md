# Gen AI / LLM / Python Interview Prep Notes

*Organized by topic below: Core LLM Concepts, RAG & Retrieval, Agents & Multi-Agent Systems, Testing/Evaluation/Monitoring, Safety & Security, Azure/Microsoft Ecosystem, Production Deployment & Cost, and Python.*

---

# Part 1: Core LLM Concepts

## Core Concepts

- **Pretraining** — teaching an AI to read by showing it billions of web pages.
- **Instruction tuning** — teaching the AI to follow human instructions.
- **Fine-tuning** — teaching the AI a specific skill for your business/use case.
- **MCP (Model Context Protocol)** — created by Anthropic; connects AI models to external data sources.
- **Agentic AI** — an AI system that takes actions, not just answers questions.

---

## Tokenization & Embeddings — Core Concepts

**Q1: What is tokenization?**

The process of breaking a string of text into smaller units called tokens (which can be whole words, sub-words, or characters, depending on the tokenizer).

**Q2: What does "contextualization" mean for embeddings?**

Each token starts with a fixed ID. That ID is passed into a model (e.g., BERT, or OpenAI's `text-embedding-3` family) and moves through the model's **attention layers**, where the model looks at surrounding words and adjusts the token's representation accordingly — so the same word gets a different vector depending on its context (e.g., "bank" near "river" vs. "bank" near "loan").

**Q3: What is the `[CLS]` token?**

A special token added to the very beginning of every input sequence in BERT-style models. As the sequence passes through the network, the attention mechanism lets the `[CLS]` token "attend to" every other token in the sequence. By the final layer, `[CLS]`'s vector effectively holds a summary representation of the entire input — which is why it's commonly used as the embedding for classification tasks or whole-sequence representations.

---

## Sampling, Generation & Context

**Q4: What is a system prompt?**

A set of instructions given to an LLM that defines how it should behave, respond, and follow rules throughout a conversation — set once at the start of a session (distinct from the user's individual messages), and typically used to establish persona, tone, constraints, and grounding rules (e.g., "answer only from the provided context").

**Q5: What is temperature?**

Controls the randomness of the model's output by scaling the probability distribution over next tokens.

- **T = 0** — effectively deterministic (always picks the highest-probability token; greedy decoding).
- **T = 1** — roughly the model's natural, unscaled probability distribution ("neutral").
- **T > 1** — flattens the distribution, making less-likely tokens more competitive → more randomness/creativity, but higher risk of incoherence.

**Q6: What is the context window?**

The number of tokens an LLM can process (input + output combined, depending on how the vendor defines it) in a single call — effectively its working memory for understanding and generating text within that request.

**Q7: What is function/tool calling?**

One of the most powerful features of modern chat completion APIs — the ability for the model to call external functions/tools (rather than just generating text), enabling it to fetch live data, take actions, or interact with external systems as part of producing a response.

**Q8: How do you reduce hallucination?**

- **Better prompting** — explicit instructions to answer only from provided context, and to say "I don't know" rather than guess.
- **Fine-tuning** — training on domain-specific data to reduce reliance on ungrounded parametric knowledge.
- **A verification/grounding layer** — RAG (grounding answers in retrieved documents), citation requirements, or a separate fact-checking pass that verifies claims against source material before returning the response.

**Q9: What is a loss function?**

A function that measures how wrong the model's predictions are compared to the actual/expected output during training — the training process works by adjusting model parameters to minimize this value.

---

## Model Selection

**Q10: How do you decide between GPT-5, Claude 4.7, and open-source Llama 4?**

| Factor | Consideration |
|---|---|
| Cost | Llama is free to host but requires GPU infrastructure |
| Quality | Claude tends to lead here |
| Latency | Smaller models (e.g., Haiku, GPT-mini) are faster |
| Customization | Only open source allows full fine-tuning; GPT and Claude charge per token |
| Data privacy | Open source can run on-premises, so data never leaves your servers |

---

## AI Engineering Roles & Tech Stack

**Q11: Data Scientist vs. ML Engineer vs. AI Engineer?**

- **Data Scientist** — analyzes data using statistics, Python, and SQL; works mostly in notebooks to generate insights.
- **ML Engineer** — deploys ML models to production; MLOps, Docker/Kubernetes, APIs, ML pipelines.
- **AI Engineer** — builds AI-powered applications; LLM selection, LangChain/LangGraph, and shipping AI products end-to-end.

**Q12: What's a typical AI Engineer tech stack?**

- **Foundation models** — OpenAI GPT-4o, Anthropic Claude, Google Gemini, Meta Llama.
- **Orchestration frameworks** — LangChain, LangGraph, CrewAI, AutoGen.
- **Vector databases** — Pinecone, Qdrant, ChromaDB.
- **Serving infrastructure** — TGI (Text Generation Inference), FastAPI, vLLM.
- **Prompt management** — Langfuse, LangSmith.
- **Evaluation** — RAGAS, LLM-as-judge.
- **Cloud AI services** — Azure OpenAI, AWS Bedrock (Google Vertex AI is the other major one worth naming).

---

## Serving Infrastructure

**Q13: How does Uvicorn handle endpoints?**

- Uvicorn is an ASGI server built on an event loop running in the main thread.
- Endpoints defined as `async def` run **directly in the event loop**, sharing the single thread with everything else — so a blocking call inside one will stall the whole server.
- Endpoints defined as plain `def` (sync) are automatically offloaded to a **threadpool**, so they don't block the event loop, but they don't get async concurrency benefits either.

---

---

# Part 2: RAG & Retrieval

## Fine-tuning vs. RAG

**Q1: When would you choose RAG over fine-tuning?**

Use **RAG** when:
- Data changes frequently
- You need to cite sources
- You need the AI to access private documents it wasn't trained on
- You want a lower-cost solution

Use **fine-tuning** when:
- Latency matters more than data freshness
- You have 1,000+ examples of input-output pairs

**Q2: When should you move from prompt engineering to fine-tuning?**

- Your prompt has grown past ~3,000 tokens trying to cover every edge case.
- Output is still inconsistent even with strong, detailed instructions.
- Long prompts are hurting latency and you need faster responses.
- You have 1,000+ high-quality input-output example pairs to train on.

---

## RAG Systems

**Q3: A RAG system is giving wrong answers in production — how do you debug it?**

- Is the right document being retrieved? Check top-k results.
- Are chunks the right size? Too small loses context; too large adds noise.
- Does the embedding model match your domain? Generic embeddings fail on medical or legal text.
- Check whether the prompt is ignoring the retrieved context.
- Check for duplicate or outdated documents polluting the results.

**Q4: How would you design a chatbot that searches across millions of documents?**

- **Smart routing** — classify whether the query even needs retrieval; some questions don't.
- **Hybrid search** — combine keyword search (BM25) with vector search to catch both exact terms and semantic meaning.
- **Reranking** — retrieve the top 100 results, then use a cross-encoder to narrow to the top 5.
- **Caching** — cache common questions.

**Q5: How do you handle a query that needs information from multiple documents?**

- **Query decomposition** — break the query into sub-questions, retrieve for each, then combine.
- **GraphRAG** — build a knowledge graph to capture relationships across documents.

---

## Chunking Strategies

**Q6: What is LangChain's `RecursiveCharacterTextSplitter`?**

It tries to split on a prioritized list of separators, from largest semantic unit down to smallest — e.g., paragraphs (`\n\n`) → lines (`\n`) → spaces → individual characters. It moves to the next, smaller separator only if the chunk is still too big.

**Q7: What's a good chunk size?**

- **200–800 tokens** is a common sweet spot for general text.
- Legal (and other dense, reference-heavy) documents often need longer chunks to preserve context like defined terms and clause structure.

**Q8: What is structure-aware chunking?**

- Recursively split by document structure — section → paragraph → sentence/token.
- Goal: each chunk should be semantically self-contained, not just a fixed character count.
- Attach metadata to each chunk (source, section title, page number, etc.) to aid retrieval and citation.

**Q9: What is semantic chunking?**

Group adjacent sentences that have high embedding similarity into the same chunk, rather than splitting on a fixed size. This keeps topically-related content together, but it's computationally expensive since it requires comparing embeddings between neighboring sentences across the whole document.

**Q10: What is the "parent-child" (parent document) retrieval pattern?**

Retrieve using small, precise child chunks (good for matching), but pass the larger parent chunk to the LLM as context (good for completeness). This balances retrieval precision with generation quality.

**Q11: What is hierarchical chunking?**

- **Child chunks**: small, precise units (e.g., 500–1,000 tokens) used for retrieval matching.
- **Parent chunks**: larger units (e.g., 5,000+ tokens, or a full section/document) used for generation context.
- Flow: split document into parent + child chunks → embed both → user query comes in → search against child chunk embeddings → find best-matching child → fetch its parent → send the parent chunk to the LLM.

---

## Vector Search & Retrieval Internals

**Q12: What is reranking in the context of vector search?**

A second-pass step that re-scores an initial set of retrieved candidates (e.g., the top 50–100 from a vector/hybrid search) using a more precise but more expensive relevance model — typically a cross-encoder that looks at the query and each document *together*, rather than just comparing precomputed embeddings. The goal is to push the truly most relevant results to the very top, since the first-pass retrieval (fast but coarser) sometimes gets close matches wrong or misses subtle relevance signals.

**Q13: Difference between hybrid search and vector search — with use cases?**

- **Vector search** — compares the semantic embedding of the query against document embeddings; finds results that are conceptually/semantically similar, even if they don't share exact keywords. *Best for:* natural-language questions, paraphrased queries, conceptual similarity ("find documents about reducing employee turnover" matching a doc that never uses the word "turnover").
- **Hybrid search** — combines vector search with traditional keyword search (e.g., BM25), merging the two ranked lists (commonly via Reciprocal Rank Fusion). *Best for:* real-world enterprise search, where queries often mix semantic intent with exact terms that must match precisely — product codes, part numbers, acronyms, person names, legal clause numbers. Vector search alone can miss an exact ID match; hybrid search catches both.

**Q14: How do you decide chunk sizes for document processing?**

- Start with a **300–800 token** range as a reasonable default, and tune based on the domain.
- **Denser, reference-heavy content** (legal, technical specs) often needs **larger chunks** to preserve necessary context (defined terms, clause structure).
- **Conversational or narrative content** can often use **smaller chunks**, since each idea is more self-contained.
- Use **overlap** between consecutive chunks (e.g., ~100 tokens) so information sitting right at a chunk boundary isn't lost.
- Prefer **structure-aware chunking** (split on section/paragraph boundaries first) over purely fixed-size splitting, so chunks stay semantically coherent rather than cutting mid-sentence or mid-table.
- Validate empirically — test retrieval quality at a couple of different chunk sizes against real queries rather than picking one number theoretically and assuming it's right.

**Q15: What embedding dimensions have you used, and can they be changed?**

Common embedding dimensionalities in practice range from a few hundred (e.g., 384, 768) up to 1,536 or 3,072 (e.g., OpenAI's `text-embedding-3-large` supports up to 3,072, and can be truncated to smaller dimensions like 256 or 1,024 via the `dimensions` parameter, trading a small amount of accuracy for lower storage cost and faster search). So yes — many modern embedding models explicitly support **reducing the output dimension** at generation time. What can't be changed after the fact is mixing dimensions/models within a single vector index — every vector in the same index must come from the same embedding model and dimensionality, so changing embedding models requires re-embedding and re-indexing all existing content.

---

## Advanced RAG Techniques

**Q16: What is HyDE (Hypothetical Document Embeddings)?**

Instead of embedding the user's raw question, ask the LLM to first generate a **hypothetical answer document** for that question. Embed that hypothetical document and use it to search the vector store — hypothetical answers tend to be closer in embedding space to real answer documents than a short question is. Retrieve the real documents this way, then generate the final answer using the original question + retrieved documents.

**Q17: What is RAG-Fusion?**

Generate several reworded variations (sub-queries) of the user's original query using an LLM, run retrieval for each variation, then combine the ranked result lists using **Reciprocal Rank Fusion (RRF)** to produce a single fused ranking — documents that rank well across multiple query variations rise to the top.

**Q18: What is Step-Back RAG (Step-Back Prompting)?**

From the original, specific question, use few-shot prompting to have the LLM generate a more general, higher-level "step-back" question. Retrieve using that broader question to pull in useful background/context, then use both the step-back context and the original question to generate the final answer.

**Q19: What is Corrective RAG (CRAG)?**

Retrieve documents, then have the LLM (or a lightweight evaluator) **score their relevance**. If relevance is above a threshold, proceed straight to generation using those documents. If it's below threshold, treat retrieval as having failed — fall back to another source (e.g., a web search) to get better context, then generate.

**Q20: What is intent-specific RAG?**

First classify the **intent** behind the user's query, then retrieve/route to the most relevant information source for that intent. Relevance scoring can use NLP techniques and user feedback signals to keep improving over time.

---

## Advanced RAG Strategies

**Q21: What are common advanced RAG strategies (beyond basic retrieve-then-generate)?**

- **Reranking** — a second-pass relevance scoring step (see cross-encoder note below).
- **Multi-step / step-back retrieval** — retrieving iteratively (breaking a query into steps) rather than in a single pass; see Step-Back RAG and query decomposition, covered earlier.
- **Agentic RAG** — an LLM/agent decides dynamically when and what to retrieve, rather than always retrieving once up front.
- **GraphRAG** — retrieval over a knowledge graph to capture relationships across documents, not just isolated chunks.
- **Contextual retrieval** — enriching each chunk with surrounding document context (e.g., a short LLM-generated summary of where the chunk sits in the larger document) before embedding, so the chunk is more self-contained and easier to match correctly.
- **Query expansion** — reformulating or adding related terms/variations to the user's query before retrieval, to improve recall.

---

## Reranking Approaches — Comparison

**Q22: When should you skip reranking?**

- Ultra-low-latency requirements, where the extra reranking pass would blow your latency budget.
- Cases that only need keyword matching (exact term/ID lookup), where semantic reranking adds no value.
- When you're resource-constrained and "good enough" retrieval quality is acceptable.

**Q23: What is cross-encoder retrieval/reranking?**

Feeds the query and a candidate document **together** into a transformer, which outputs a single joint relevance score. Highest accuracy of the reranking approaches, but too slow to run against your whole corpus — it's used as a **reranking** step on a small candidate set (e.g., top 50–100 from an initial cheap search), not as the primary retrieval method itself.

**Q24: What is ColBERT (as a reranking option)?**

Encodes the query and document into **one vector per token** (rather than one vector for the whole document), then checks each query token against every document token to find the best match, and sums those best-match scores.

- **Pro:** meaningfully more accurate than single-vector dense retrieval, since it preserves fine-grained, token-level detail.
- **Con:** significant storage overhead (many vectors per document instead of one) and higher infrastructure complexity — it's **slower and heavier than standard single-vector dense retrieval**, though still faster than running a full cross-encoder over the whole corpus. Its real position: **more accurate than plain dense retrieval, faster than a cross-encoder** — a middle ground, not "extremely fast" in absolute terms.

**Q25: What is LLM-based reranking?**

Use a generative model itself as the judge — give it the retrieved candidate documents and the query, and ask it to reason about (or directly rank/generate from) the most relevant ones.

- **Pro:** strong reasoning ability — can weigh relevance in nuanced, context-sensitive ways a similarity score can't.
- **Con:** high latency and cost, since it's a full LLM call rather than a lightweight scoring model.

---

## RAG Evaluation Metrics

- **Faithfulness** — is the answer grounded in the retrieved documents? If faithfulness < 0.85, investigate hallucination.
- **Answer relevance** — does the response actually answer the question?
- **Context precision** — are the retrieved documents relevant? If < 0.7, fix retrieval.
- **Context recall** — did retrieval surface all the relevant documents?

**Q26: How do you automatically detect hallucination in production?**

- Faithfulness scoring with RAGAS.
- Ask the LLM the same question multiple times and check consistency.
- Ask the LLM to cite sources, then verify those sources.
- Use a second model to verify the first model's answer.

---

## RAG Evaluation Metrics — Precision & Recall

**Q27: Context precision formula?**

`Precision = True Positives / (True Positives + False Positives)`

Of everything the system *retrieved* (predicted as relevant), how much was actually relevant?

**Q28: Context recall formula?**

`Recall = True Positives / (True Positives + False Negatives)`

Of everything that was *actually* relevant, how much did the system successfully retrieve?

**Q29: What is coherence (as an eval metric)?**

Measures whether the model's output flows smoothly and reads naturally, human-like — not just factually correct, but well-structured and readable.

---

## RAG System Design (End-to-End Architecture)

**Q30: How would you design a production RAG system end-to-end?**

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

## Production RAG Design

**Q31: Design a RAG system for a company with 100k documents.**

1. **Ingestion pipeline** — document parsing (e.g., BeautifulSoup for HTML), intelligent/semantic chunking, metadata extraction, embedding generation, vector store selection.
2. **Retrieval strategy** — hybrid search, reranking.
3. **Query processing** — context compression, citation generation, faithfulness checking.
4. **Monitoring** — RAGAS metrics, user feedback, retrieval quality dashboards.

**Q32: Design a multi-tenant AI platform where different customers have isolated data and models.**

1. **Data isolation** — separate vector store/collection per tenant.
2. **Prompt isolation** — tenant-specific system prompts.
3. **Model isolation** — potentially different models/fine-tunes per tenant.
4. **Observability isolation** — each tenant sees only their own usage metrics and cost tracking.

**Q33: How do you increase LLM response speed?**

1. **Semantic caching** — often 30–40% of queries are duplicates or near-duplicates; an instant cached response means zero additional LLM cost.
2. **Model routing** — e.g., ~70% of queries to a cheap model, ~25% to a medium model, ~5% to your most capable (and expensive) model.
3. **Async processing** — push non-real-time jobs to a queue so they don't block the user.
4. **Rate limiting + circuit breaking** — prevent cascading failures.
5. **Auto-scaling.**
6. **Multi-region deployment.**

**Q34: Common RAG failure modes?**

1. **Retrieval failure** — wrong chunks retrieved. *Fix:* hybrid search, better chunking strategy.
2. **Faithfulness failure** — the LLM ignores retrieved chunks and answers from its own training knowledge instead. *Fix:* RAGAS faithfulness monitoring, guardrails.
3. **Context window overflow** — too much retrieved context causes the model to "lose" information in the middle (the well-documented "lost in the middle" effect).
4. **Stale knowledge** — the knowledge base isn't updated when source documents change. *Fix:* document freshness monitoring.
5. **Query-document mismatch** — the user asks in a different style/phrasing than the documents are written in. *Fix:* query rewriting.

**Q35: What is HyDE, and when do you use it to improve RAG?**

HyDE solves a fundamental mismatch in RAG: user queries are short, but knowledge documents are long-form answers — so directly embedding the query and comparing it to document embeddings often works poorly.

**Solution:** take the user's query, use an LLM to generate a *hypothetical* answer, embed that hypothetical answer (not the raw query), and search the vector DB using that embedding — then return the real matching documents. It works because the hypothetical answer is written in the same style as real documents, so vector similarity matching performs better than trying to match a short question directly against long-form content.

**Q36: How do you handle multi-hop questions in RAG (requiring information from multiple documents)?**

1. **Iterative retrieval** — answer a sub-question, use that answer to inform the next retrieval, repeat until all hops are resolved.
2. **Sub-question decomposition** — an LLM breaks a complex query into sub-questions, answers each separately, then synthesizes a final answer.
3. **Knowledge graph RAG** — represent documents as a graph and traverse edges to find connected information across documents.
4. **Summarize-then-index** — index both individual chunks *and* whole-document summaries; use the summaries to identify which documents are relevant, then drill into their chunks for detail.

---

## Caching Strategy

**Q37: How should you design caching for a RAG/LLM system?**

- If the LLM (or a downstream dependency) is down, serving a **cached response** is better than a hard failure.
- **Semantic caching** — if the current query is similar enough (by embedding similarity) to a previously cached query, serve the cached response instead of re-running the full pipeline — saves cost and latency.

**Typical flow:**
```
User query → query cache check → embedding cache → vector search → LLM call → store result in cache
```

---

## Model Routing

**Q38: What is model routing?**

Route simple queries to a smaller/cheaper/faster model, and route complex queries to a more capable (and more expensive) model — rather than sending every request to your most powerful model by default. This is one of the main cost-reduction levers covered earlier (see "LLM bill hit ₹50 lakh/month" above).

---

---

# Part 3: Agents & Multi-Agent Systems

## Agents

**Q1: How do you handle an agent that gets stuck in an infinite loop?**

- Set an iteration limit — max 10–15 steps per task.
- Set a budget limit — max tokens or cost per task.
- Add a supervising agent to monitor and intervene.

**Q2: A Gen AI project failed in production — what do you do?**

- Get context — what was being built?
- Check hallucinations, cost, latency, and user behavior.

---

## Agent Design & Patterns

**Q3: What is agentic AI (why do we need agents)?**

Common use cases:
- Replacing static web forms (lead generation, inquiry forms) with a conversational interface.
- Replacing website search/navigation — the agent answers directly instead of making the user hunt for information.
- Routing inquiries to the right person or department in an organization.

**Q4: How do you plan/design an agent?**

- Define the agent's **scope** and **purpose**.
- Decide which **channels** it will be deployed on (web, Teams, etc.).
- Write clear, specific agent instructions.
- Favor **single-responsibility** agents over one agent trying to do everything.
- Clearly define: who the agent is, what it should do, and how it should behave.

**Q5: What is agent metacognition?**

The agent is "aware" of its own internal process — able to monitor its own performance, identify areas for improvement, and adapt its behavior accordingly.

**Q6: How do you build a proper agent tool?**

- Use a decorator (e.g., `@tool`) to register the function as a tool.
- Add proper type annotations so the model knows expected inputs/outputs.
- Include proper error handling.
- Return data in a form the model can actually use in its response.
- Keep each tool scoped to **one specific task**.
- Use structured output (e.g., Pydantic models) for reliability.
- Tools are for scenarios requiring dynamic interaction with an external system (APIs, databases, etc.) — not for things the model can just reason through itself.

**Q7: What is Responsible AI?**

The notes only mention one principle — **transparency** (making sure people understand they're interacting with an AI agent, not a human). Microsoft's Responsible AI framework actually defines six core principles: **fairness, reliability & safety, privacy & security, inclusiveness, transparency,** and **accountability**.

### Multi-Agent Patterns

**Q8: What is a multi-agent system?**

A design pattern where multiple agents work together toward a common goal — useful for large or complex tasks that benefit from division of labor (e.g., large-scale data processing).

**Q9: What is the group chat multi-agent pattern?**

Each agent participates in a shared group chat like a "user" would, exchanging messages via a common messaging protocol. Agents can post to the chat, read messages from it, and respond to other agents' messages. Common use cases: customer support, task management, workflow automation.

**Q10: What is the collaborative filtering agent pattern?**

Multiple agents collaborate to generate recommendations for users — useful in domains like stock market or investment recommendations, where different agents might specialize in different signals or strategies.

---

## Agent Memory, Patterns & LangGraph

**Q11: What is agent memory? Explain the 4 types.**

*(One common framework — worth noting some sources describe this slightly differently, e.g., adding a separate "semantic memory" for general facts.)*

- **Sensory memory** — the current message/context within the immediate turn.
- **Short-term memory** — the full conversation history within the current context window; limited by the model's context length.
- **Long-term memory** — stored externally (a database), persisting beyond any single conversation.
- **Episodic memory** — summaries of past sessions, stored externally, so the agent can recall "what happened last time" without re-reading the full transcript.

**Q12: What is Human-in-the-Loop (HITL)?**

The agent requires human approval before taking **irreversible** actions (sending an email, making a payment, etc.). Also used for **output review** — e.g., an agent drafts a report or email, and a human approves it before it's published/sent.

**Q13: AI workflow vs. AI agent?**

- **AI workflow** — predefined, fixed sequence of steps; the path is deterministic and known in advance.
- **AI agent** — the model decides its own steps dynamically based on reasoning; the path is non-deterministic and can vary run to run.

**Q14: ReAct vs. Plan-and-Execute vs. Reflection agent patterns?**

- **ReAct** (Reason + Act) — makes one decision at a time based on the current state; good for customer support and other interactive, adaptive tasks.
- **Plan-and-Execute** — a planner LLM creates the full task breakdown up front, then an executor carries out each step; good for report generation, data analysis — tasks where the full plan can reasonably be known ahead of time.
- **Reflection agent** — generates a response, critiques its own output, then revises — looping until a quality threshold is met; good for code generation, writing tasks.
- **Hybrid** — use ReAct within each step of a Plan-and-Execute pipeline.

**Q15: What is the supervisor pattern (multi-agent)?**

A supervisor agent routes tasks to sub-agents dynamically — best when the task has unclear/variable steps and needs content-based dynamic routing rather than a fixed pipeline.

**Q16: LangGraph — core building blocks?**

- **State graph** — the shared state, typically a typed dictionary.
- **Node** — a Python function representing one step.
- **Edges** — define how execution moves between nodes.

**Q17: What is checkpointing (in LangGraph)?**

Saving the complete state of execution at each step, so a run can be replayed, resumed, or forked later.

- **Fault tolerance** — if the agent crashes at step 15 of 25, it can resume from step 15 rather than starting over.
- **HITL support** — execution pauses at a human-approval step, and state persists until the human responds (even if that takes hours or days).
- Every agent decision and state transition is recorded, enabling **time-travel debugging** (stepping back through prior states to see exactly what happened).

**Q18: What is the `Command` object in LangGraph?**

Lets a node do two things in a single return value: **update the shared state** and **decide which node to call next** — combining state update and routing/control-flow decisions in one step, instead of needing separate mechanisms for each.

```python
return Command(
    update={"status": "completed"},
    goto="next_node"
)
```

**Q19: When should you use a single agent vs. multiple agents?**

**Use a single agent when:**
- The solution handles one main task or intent.
- A single (or small) team manages the whole application.
- No separate authentication, access control, or deployment is needed per component.
- Tool usage is relatively simple, and there's no need to reuse the agent elsewhere.

**Use multiple agents when:**
- Different parts of the task require different specialized expertise or tools.
- Agents need separate authentication/access control from each other.
- Different teams manage and publish different components independently.
- Agents need to be reused across multiple applications.
- As a rough practical signal: if a single agent's tool-call chain is regularly running into the **30–40+ tool call** range for one task, that's often a sign the task should be decomposed across multiple specialized agents instead.

**Q20: What is Spec-Driven Development (for AI coding agents)?**

A workflow for directing a coding agent that starts with a detailed specification *before* any code gets written, broken into three steps:

1. **Specify** — a high-level description of what's being built and why: the user journey, the experience, the final result, who it's for, and what problem it solves.
2. **Plan** — give the coding agent the desired stack, architecture, constraints, and performance targets; internal documentation can be provided here too, so the agent can integrate it directly into the plan.
3. **Tasks** — break the spec and plan into concrete, actionable work items; each task should be small enough to implement and test in isolation.

**Q21: What does a multi-agent architecture look like in LangGraph specifically?**

Each agent typically has its own prompt, its own tools, and its own memory/state, collaborating with other agents toward a shared goal. In LangGraph's graph model:
- **Agent = Node** — each agent is represented as a node in the graph.
- **Communication path = Edge** — connections between agents are represented as edges, defining how control/data passes between them.

Key design questions: what are the independent agents needed for this task, and how should they be connected? Grouping tools/responsibilities sensibly, and giving each agent its own focused prompt (rather than one giant prompt trying to cover everything), tends to produce better results than one overloaded agent.

**Q22: What is LangSmith?**

A platform (from the LangChain team) for tracing, debugging, evaluating, and monitoring LLM applications in production.

**Q23: How do you design a multi-agent system when agents have different capabilities/models?**

*(Caveat: specific model recommendations age quickly — treat the pattern as the durable part, and re-check current best-fit models before using these exact names in an interview.)*

- **Router agent** — fast, cheap model (e.g., a "flash"/"mini" tier) for quick classification.
- **Research agent** — a model with a large context window, for reading/synthesizing long documents.
- **Analysis/code agent** — a strong reasoning model.
- **Writing agent** — a strong generation model.
- **Review/QA agent** — fast, cheap model for quick checks.
- **Orchestrator** — a capable model handling complex routing decisions across the whole system.

**Q24: Multi-agent anti-patterns?**

1. **Over-agentification** — using an agent for a task that's actually deterministic and doesn't need one.
2. **Infinite loops** — no max step count, no timeout, no cost limit.
3. **Shared state without a lock** — multiple agents writing to shared state concurrently, corrupting it (a classic race condition).
4. **Too many agents** — adds unnecessary latency for tasks that didn't need that much decomposition.
5. **No fallback** — if the primary agent fails, the whole system fails, with nothing to degrade to gracefully.

---

## Agent Reliability & Error Handling

**Q25: How do you handle tool errors in a production agent?**

Production agents face tool failures constantly — API timeouts, rate limits, invalid data returned.

1. **Structured errors, not raised exceptions** — return something like `{success: false, error: "rate_limit", retry_after: 10}` instead of throwing, so the agent can read the error type and decide what to do next.
2. **Retry logic** — exponential backoff for transient errors (rate limits, timeouts), capped at a small number of retries (e.g., max 3).
3. **Graceful delegation** — surface an LLM-generated, human-readable error message rather than a raw stack trace.
4. **Step/cost limits** — "error budget" and "max hard limit" are two different controls:
   - **Error budget** — how many *failed* tool calls/errors the agent tolerates before giving up or escalating to a human (e.g., stop after 3 consecutive tool failures).
   - **Max step hard limit** — a cap on the *total* number of steps the agent can take overall, regardless of success or failure (e.g., never exceed 50 steps in one run).

**Q26: How do you prevent an AI agent from entering an infinite loop?**

*(This expands on the shorter version covered earlier — same core idea, more production detail.)*

1. **Max step limit** — e.g., `max_steps = 25`; terminate and alert if exceeded.
2. **Budget cap** — track cost per session; terminate if `session_cost > $2` (or whatever threshold fits your economics).
3. **Repeat-action detection** — if the last 3 tool calls are identical, the agent is likely looping; detect and terminate with a clear reason (e.g., `repeat_action_detected`).
4. **Per-step timeout** — each tool call must complete within a fixed time (e.g., ≤30 seconds) or gets cancelled — prevents the agent from hanging forever on an unresponsive API.
5. **Restrict to approved tools only.**

---

## LangGraph — Interrupt Node

**Q27: What does an interrupt node do in LangGraph?**

Pauses graph execution at that point and saves the current state to a **checkpointer**, so execution can be resumed later — commonly used for human-in-the-loop workflows (e.g., pause and wait for human approval before continuing to the next step).

---

## LangChain / LangGraph Architecture & Lifecycle

**Q28: Explain the lifecycle of a LangChain-based AI project.**

1. **Requirement gathering** — what's the task, who are the users, what data sources are involved.
2. **Data ingestion** — load and parse source documents (PDFs, SharePoint, databases, APIs).
3. **Chunking & embedding** — split documents into chunks, generate embeddings, store in a vector database.
4. **Chain/agent design** — decide whether the task needs a simple chain (fixed steps) or an agent (dynamic tool use), and build it using LangChain's components (prompts, LLM wrappers, retrievers, tools).
5. **Prompt engineering & tuning** — iterate on system prompts, few-shot examples, and output format until responses are reliable.
6. **Evaluation** — test against a golden dataset (accuracy, faithfulness, relevance).
7. **Guardrails & safety** — add input/output filtering, PII handling, hallucination checks.
8. **Deployment** — wrap in an API (commonly FastAPI), containerize, deploy to cloud infrastructure.
9. **Monitoring & observability** — tracing (e.g., LangSmith), logging, cost/latency dashboards.
10. **Iteration** — use production feedback and failure cases to keep improving prompts, retrieval, and evaluation coverage.

**Q29: What is LangGraph, and how is it different from LangChain?**

- **LangChain** — a library of composable building blocks (prompts, chains, retrievers, tools) primarily suited to linear or lightly-branching pipelines.
- **LangGraph** — built on top of LangChain's components, but models the application as an explicit **graph** of nodes and edges, supporting **cycles/loops**, **persistent state** (checkpointing), and **human-in-the-loop** pauses — which a simple LangChain chain doesn't natively support. LangGraph is the better fit once your application needs an agent that can loop, branch dynamically, or be paused/resumed (e.g., waiting on human approval).

**Q30: Difference between a Chain, an Agent, and a Tool in LangChain?**

- **Chain** — a fixed, predefined sequence of steps (e.g., prompt → LLM → parser) — the path is deterministic and known ahead of time.
- **Agent** — an LLM that decides its own sequence of actions dynamically, typically by reasoning about which tool to call next based on the current state — the path is not fixed in advance.
- **Tool** — a specific capability the agent (or a chain) can invoke — a function, an API call, a database query, a search function, etc. Tools are the "actions" an agent chooses from; the agent is the decision-maker choosing which tools to use and when.

---

## Multi-Agent Systems in Practice

**Q31: Have you created chatbots or multi-agent systems? Describe the design.**

*(Model answer framework — describe your own system using this shape.)* A strong answer covers: the overall architecture (single agent vs. multi-agent, and why), the orchestration framework used (LangGraph, Semantic Kernel, Microsoft Agent Framework, etc.), how state/memory was managed across turns, which tools/external systems the agent(s) could call, how errors and edge cases were handled, and how the system was evaluated and monitored in production.

**Q32: How do you ensure correct agent communication in a multi-agent setup?**

- Define **clear, structured message formats** between agents (e.g., a shared, typed state schema) rather than free-form text handoffs — reduces ambiguity about what's actually being passed.
- Use an explicit **orchestrator/supervisor** to route and sequence communication, rather than letting agents communicate in an unconstrained, ad hoc way.
- **Guard shared state** against concurrent writes (locking or a single writer per state slice) to avoid corruption.
- Log every inter-agent message/handoff for traceability, so a miscommunication can actually be debugged after the fact.
- Test agent-to-agent handoffs explicitly as part of your evaluation suite, not just end-to-end outcomes.

**Q33: For some scenarios, an agent isn't working correctly when fetching details from an external connector — how do you diagnose this, and who do you escalate it to?**

1. **Isolate the layer** — is the failure in the connector/API itself (timeout, auth failure, bad response), in how the agent is calling it (wrong parameters), or in how the agent is interpreting the result?
2. **Check structured error output** — a well-built connector should return a structured error (not just an exception), making it clear whether it's a transient issue (retry) or a real fault.
3. **Reproduce in isolation** — call the connector directly outside of the agent to confirm whether the issue is connector-side or agent-side.
4. **Escalation:** if it's the connector/API itself misbehaving (wrong data, downtime, changed schema) → escalate to the team owning that integration/API. If it's the agent's own logic/prompt/tool-calling behavior → that's on the AI/agent development team to fix directly.

**Q34: How do you ensure the data returned by an AI chatbot is accurate? What methodology do you follow?**

Ground responses in retrieved source data (RAG) rather than relying on model memory; add faithfulness checks against that source data; maintain a **golden evaluation set** with expert-verified correct answers; run automated evaluation (RAGAS-style metrics, LLM-as-judge) plus periodic human review; and treat every production accuracy failure as a candidate to add to the regression test set, so the same mistake gets caught automatically going forward.

**Q35: A critical production issue affects 1% of cases — what do you do?**

1. **Assess severity and blast radius first** — even at 1% of volume, is the *impact* of those failures severe (e.g., wrong medical/financial info) or low-stakes? Severity, not just frequency, determines urgency.
2. **Reproduce and isolate** — pull the failing cases' logs/traces, identify what they have in common (a specific query pattern, a specific tool, a specific data source).
3. **Contain** — if needed, add a targeted guardrail or fallback for that specific failure pattern while a proper fix is developed, rather than leaving it live.
4. **Fix and validate** against the golden/regression set, specifically including the newly discovered failure cases.
5. **Add monitoring** so a recurrence of that specific pattern is caught immediately, not rediscovered independently later.

**Q36: Is there a custom agent where you performed testing? Have you run evaluations effectively — what steps make evaluation more effective?**

*(Model answer framework.)* Describe a specific agent you tested, and the evaluation steps that made it effective: a **golden dataset** with diverse, expert-verified examples (including edge cases and adversarial/trick queries); automated metrics (RAGAS-style faithfulness/relevance for RAG, trajectory/tool-accuracy checks for agents); LLM-as-judge for open-ended quality dimensions, calibrated periodically against human ratings; and treating the evaluation set as a **living dataset** — adding every real production failure to it over time, so evaluation coverage keeps growing rather than staying static.

**Q37: How did you manage access control?**

*(Model answer framework.)* Typical layers: authentication (e.g., Azure AD/Entra ID, API keys, or JWT tokens depending on the system), authorization checks tied to user roles/claims (e.g., filtering retrieved documents by the requesting user's access level — relevant if using Azure AI Search with document-level metadata for role-based filtering), and, for agent tool access specifically, restricting which tools/actions a given agent or user context is allowed to invoke.

**Q38: Where did you store data during the application's implementation stage?**

*(Model answer framework.)* Describe your actual setup — commonly: raw source documents in Blob Storage, structured/application data in a relational DB or Cosmos DB, vector embeddings in a vector database or Azure AI Search's vector index, and secrets in Azure Key Vault (never in application config files committed to source control).

**Q39: Can you explain your understanding of Microsoft Agent Framework? Have you built an agent-based solution with it? How would you design a multi-step AI agent system (chatbot + APIs + database)?**

**Microsoft Agent Framework (MAF)** is Microsoft's open-source SDK/runtime for building AI agents and multi-agent workflows, with consistent APIs across .NET and Python. It reached general availability in 2026, converging what were previously two separate efforts — **AutoGen** (multi-agent patterns/research) and **Semantic Kernel** (enterprise features like session state, type safety, telemetry) — into a single supported framework. It provides chat clients, tool/MCP integration, middleware, and explicit multi-step **workflows**, plus a built-in **agent harness** (the runtime scaffolding — tool-calling loop, memory, context management, human-in-the-loop approvals — that turns a raw model into an agent that can actually act). For production, agents built with it can be deployed as **Hosted Agents** in Microsoft Foundry Agent Service, which packages the agent as a container with built-in identity, auto-scaling (including scale-to-zero), and observability.

**Designing a multi-step agent system (chatbot + APIs + database):**

1. **Entry point** — a chat interface (web/Teams/API) receives the user message.
2. **Orchestrator/router** — classifies intent and decides which path/agent handles the request.
3. **Retrieval layer** — for knowledge questions, retrieve from a vector store/Azure AI Search.
4. **Tool layer** — for actionable requests, the agent calls specific tools that wrap your REST APIs (e.g., "create order," "check inventory") and/or query the database directly through a data-access tool.
5. **State/memory layer** — conversation history and any multi-step task state persisted across turns (e.g., via a checkpointer, as in LangGraph, or MAF's session state management).
6. **Guardrails** — input/output safety checks, and human-approval gating before any irreversible action (e.g., placing an order, sending a payment).
7. **Response generation** — the LLM composes a final natural-language response, grounded in whatever was retrieved/returned from tools and the database.
8. **Observability** — trace every step (which tool was called, what the DB returned, what the model generated) for debugging and evaluation.

---

## Chatbot & Agent Internals

**Q40: What happens in the background when a chatbot receives a simple question?**

1. The user's message is received by the application/API layer.
2. **Session/context lookup** — retrieve the ongoing conversation history (if any) for that user/session.
3. **Guardrail checks** — input filtering (prompt injection detection, PII scanning) before anything is sent to the model.
4. **Retrieval (if RAG-based)** — the query is embedded, relevant documents/chunks are retrieved (often hybrid search + reranking).
5. **Prompt assembly** — the system prompt, conversation history, retrieved context, and the new user message are assembled into the final prompt sent to the LLM.
6. **LLM call** — the model generates a response (possibly streamed back token by token).
7. **Output guardrails** — checking the response for hallucination/faithfulness, PII, harmful content, before it's shown to the user.
8. **Logging & state update** — the exchange is logged (for monitoring/evaluation) and the conversation state/session is updated with the new turn.
9. The response is returned to the user.

**Q41: What are the main components of an "AI engine" (a production LLM application)?**

- **LLM/foundation model layer** — the model(s) actually doing generation/reasoning.
- **Orchestration layer** — chains/agents/workflows coordinating steps (e.g., LangChain/LangGraph).
- **Retrieval layer** — vector database + retrieval/reranking logic, for RAG-based systems.
- **Tool/integration layer** — connectors to external APIs, databases, and systems the agent can act on.
- **Memory/state layer** — conversation history, session management, long-term memory storage.
- **Guardrails layer** — input/output safety, PII handling, hallucination checks.
- **Observability layer** — logging, tracing, metrics, evaluation pipelines.
- **API/serving layer** — the interface (typically REST, e.g., FastAPI) through which applications actually call the system.

**Q42: How would you prevent hallucination in a chatbot's responses?**

- Ground responses in retrieved context (RAG), and explicitly instruct the model in the system prompt to answer only from the provided context — and to say "I don't know" rather than guess when the context doesn't contain the answer.
- Add a **faithfulness check** — verify (via a separate model call, or an automated metric like RAGAS) that claims in the response are actually supported by the retrieved documents.
- Require **citations** — ask the model to cite which source/chunk supports each claim, making unsupported claims easier to spot (by a human or an automated checker).
- Lower **temperature** for factual tasks, reducing unnecessary creative drift.
- Use a **verification/self-check layer** — a second pass where the model (or another model) checks its own answer against the source material before returning it.
- Continuously feed real production hallucination cases back into your regression/eval test suite.

**Q43: Is chat session management manual or automated?**

In a well-built system, it's largely **automated**: a session ID is generated (or supplied by the client) when a conversation starts, and the conversation history/state is automatically persisted and retrieved by session ID on each turn — typically backed by a database or cache (Redis, a session table, or LangGraph's checkpointer for agentic systems). Manual intervention usually only comes in for things like session expiry policies, manually clearing/resetting a session, or handling edge cases like session handoff between systems.

**Q44: How should an agent handle repeated, identical user queries? Does this impact cost?**

Yes — repeated/near-duplicate queries are a direct cost lever. The standard approach is **semantic caching**: check whether the current query is similar enough (by embedding similarity, not just exact string match) to a recently cached query, and if so, serve the cached response instead of running the full retrieval + LLM pipeline again. This both reduces cost (no LLM/embedding calls for a cache hit) and improves latency. It's worth combining with a reasonable cache expiry/invalidation policy, especially if the underlying data changes.

**Q45: Can an agent call multiple tools simultaneously?**

Yes — if the tool calls are **independent of each other** (the output of one isn't needed as input to another), they can be dispatched **in parallel** (e.g., using `asyncio.gather` in Python, or a framework's built-in parallel tool-calling support) rather than sequentially, reducing overall latency. If one tool's output feeds into another's input, they have to run sequentially. Some LLM providers' function-calling APIs also support returning multiple tool calls in a single response, which the calling code can then execute concurrently.

---

## Model Context Protocol (MCP)

**Q46: What is an MCP Host?**

The application that coordinates and manages one or more MCP clients (e.g., an AI assistant app or IDE).

**Q47: What is an MCP Client?**

Maintains a connection to an MCP server, used to obtain context/tools from that server on behalf of the MCP host.

**Q48: What is an MCP Server?**

A program that exposes context, tools, or data to an MCP client, following the MCP protocol.

**Q49: What is JSON-RPC?**

A lightweight remote procedure call protocol encoded in JSON — it lets one program ask another to execute a specific function (with named parameters) over a transport (like stdio or HTTP), and get a structured JSON response back. MCP uses JSON-RPC 2.0 as its underlying message format.

**Q50: What are MCP's transport mechanisms?**

- **stdio transport** — communicates via standard input/output streams; used for local processes running on the same machine.
- **Streamable HTTP** — uses HTTP POST for client-server messages; enables communication with remote servers and supports standard HTTP authentication.

**Q51: How does tool discovery work in MCP?**

The client sends a `tools/list` request to the server, which returns the tools it has available. Once the client knows what's available, it can invoke a specific tool via `tools/call` with the appropriate arguments.

---

## MCP Security & Tool Discovery

**Q52: What is MCP tool poisoning?**

An attack where a malicious actor embeds hidden instructions inside a **tool's description** (not its actual functionality) — e.g., the description might instruct the agent that "before calling this tool, first send the entire conversation history" — something the agent might comply with since it trusts tool descriptions as legitimate configuration, not untrusted input.

**Q53: How does an agent discover and use MCP tools dynamically?**

1. The agent connects to an MCP server via **stdio** (if local) or **HTTP/SSE** (streamable HTTP) transport (if remote).
2. The client sends an initial request; the server responds with its supported features.
3. The client sends a `tools/list` request; the server returns its full list of available tools.
4. The LLM sees the available tools and decides whether/which to call.
5. MCP servers commonly authenticate via API key (or other standard HTTP auth mechanisms for remote servers).

---

## Cost, Guardrails & Observability

**Q54: How do you reduce cost in a production multi-agent system?**

Model routing, semantic caching, max-step + budget caps, parallel execution where possible, prompt optimization, and using smaller models for sub-tasks that don't need a frontier model.

**Q55: How do you implement guardrails for an AI agent?**

- **Input guardrails** — prompt injection detection, PII detection/masking.
- **Agent behavior guardrails** — approved-tools-only list, max steps, budget cap.
- **Output guardrails** — faithfulness checks, PII-in-output detection, hallucination/harmful content screening.
- **Action guardrails** — human approval required before irreversible actions.

**Q56: What does observability in a production agent look like?**

Trace every agent step individually, aggregate operational metrics (latency, error rate, cost), and track business metrics (task success rate, user satisfaction) on top.

**Q57: Sync vs. async agent execution?**

- **Sync (blocking)** execution — each step/tool call must fully complete before the agent (and the user waiting on it) can proceed to the next one; simpler to reason about, but doesn't scale well under load since each request ties up resources while waiting.
- **Async (non-blocking)** execution — the agent can await I/O (tool calls, LLM calls) without blocking a thread, allowing many concurrent user sessions and parallel tool calls (e.g., calling several independent tools at once instead of sequentially) — essential for production throughput at scale.

**Q58: High-availability architecture for agents in production?**

- Multi-agent (and multi-instance) deployment for redundancy.
- **LLM provider failover** — fall back to a secondary provider/model if the primary is down.
- **Tool circuit breakers** — each tool gets its own circuit breaker; if a tool's failure rate crosses a threshold, the circuit "opens" and the system falls back to a cached/last-known-good response instead of continuing to call a failing tool.
- **Graceful degradation** — reduced functionality rather than total failure when a dependency is down.

**Q59: How do you evaluate and debug agent trajectories?**

- **Trajectory evaluation** — did the agent follow the correct sequence of steps/tool calls to reach the answer, not just whether the final answer happened to be right?
- **Tool accuracy** — did it call the correct tools, with correct arguments?

**Q60: How do you debug a failing agent?**

Reproduce the failure → isolate the specific failing component → reproduce it in isolation (e.g., using LangGraph's time-travel/checkpoint replay) → add that case as a permanent regression test.

**Q61: What is LLM-as-judge bias, and what are the common biases?**

Since an LLM is being used to *evaluate* another LLM's output, the judge itself can introduce systematic biases:

- **Position bias** — favoring whichever answer appears first (or last) when comparing two outputs, regardless of actual quality.
- **Verbosity bias** — favoring longer, more detailed-looking answers even when they aren't more correct.
- **Self-preference bias** — a model rating outputs more favorably when they're stylistically similar to its own outputs (e.g., a model from the same family as itself).

**Mitigations:** randomize the order of compared outputs, control/normalize for response length, use multiple different judge models and cross-check, and periodically calibrate judge scores against human-labeled examples.

**Q62: What does CI/CD look like for an agent application?**

1. Unit tests.
2. Integration tests.
3. Golden dataset evaluation (run against your curated golden set before deploying).
4. Regression checks (make sure nothing that used to work broke).
5. **Canary deployment** — e.g., route 5% of traffic to the new version, 95% to the current stable version, and monitor before a full rollout.

---

## Production Agentic AI — System Design Deep Dives

**Q63: Explain an end-to-end agentic system design — including planner agents, tool orchestration, state management, memory, retry strategies, and human approval workflows.**

**1. Entry & routing layer**
User request comes in (chat, API, event trigger) → a lightweight router/classifier decides whether this needs simple retrieval, a single tool call, or full multi-step planning.

**2. Planner agent**
For complex requests, a planner LLM breaks the goal into a sequence (or graph) of sub-tasks up front — e.g., "check inventory → calculate shipping → create order" — rather than deciding one step at a time. This gives predictability and lets you validate the plan before execution starts (useful for higher-stakes workflows).

**3. Tool orchestration**
An executor works through the plan, calling tools (APIs, DB queries, search, calculations). Independent steps are dispatched in parallel where possible; dependent steps run sequentially. Each tool call goes through a consistent wrapper that handles timeouts, structured error returns, and logging — not raw, unhandled exceptions.

**4. State management**
A shared, typed state object (e.g., a LangGraph state graph, or an equivalent structured session object) tracks: the original goal, completed steps, intermediate results, and any pending approvals. State is checkpointed after each step, not just held in memory — this is what makes steps 6–7 below actually possible.

**5. Memory**
- **Short-term** — the current task's state and conversation context (bounded by the context window).
- **Long-term** — persisted externally (a database), so context can survive across sessions (e.g., user preferences, prior interactions).
- **Episodic** — summarized records of past sessions, for recall without replaying full transcripts.

**6. Retry strategies**
- Transient errors (timeouts, rate limits) → automatic retry with **exponential backoff**, capped at a small number of attempts (e.g., 3).
- Non-retryable errors (invalid input, permanent 4xx-type failures) → fail fast, don't waste retries.
- A step that keeps failing after retries → surfaced as a structured failure to the planner, which can choose an alternate path or escalate to a human, rather than the whole run silently dying.

**7. Human-approval workflow**
Before any **irreversible** action (sending money, sending an external email, deleting data), execution pauses, the state is checkpointed, and a human is prompted to approve/reject. Because state is checkpointed (not just held in a live process), this pause can last minutes, hours, or days without losing progress — execution resumes exactly where it left off once a decision comes back.

**8. Guardrails wrapped around all of the above**
Input validation, output faithfulness/safety checks, and hard limits (max steps, cost budget, time budget) to prevent runaway execution — covered in more detail in the guardrails and infinite-loop sections earlier in this doc.

---

**Q64: Architecture decision-making — when do you use a single agent, a multi-agent architecture, or a workflow framework like LangGraph or Semantic Kernel? Justify by complexity, scalability, and observability.**

| Factor | Single agent | Multi-agent | Workflow framework (LangGraph / Semantic Kernel / Microsoft Agent Framework) |
|---|---|---|---|
| **When to choose** | One clear intent/task, simple tool use, no need for specialized sub-roles | Task naturally decomposes into specialized roles (research, coding, writing, review) needing different prompts/tools/models | Any of the above, once you need explicit control flow: branching, loops, checkpointing, human-in-the-loop, or long-running state |
| **Complexity** | Lowest — one prompt, one tool set, straightforward to reason about | Higher — must design agent-to-agent communication, shared state, and avoid race conditions on that state | Framework absorbs a lot of this complexity for you (state, routing, retries) at the cost of a steeper initial learning curve |
| **Scalability** | Limited — a single agent doing everything tends to hit context/latency/cost limits as scope grows | Scales better for breadth (more specialized tasks) but adds coordination overhead and latency if overused | Scales well operationally — frameworks provide built-in patterns (parallel branches, checkpointing) that would otherwise be hand-rolled and error-prone |
| **Observability** | Simplest to trace — one execution path | Harder — need to trace across agent boundaries, not just within one agent | Frameworks typically ship first-class tracing/checkpointing (e.g., LangGraph's time-travel debugging, MAF's built-in telemetry), which is a major practical reason to adopt one once things get complex |

**Rule of thumb:** start with a single agent. Move to multiple agents only when a single agent's responsibilities are genuinely separable and would each benefit from a distinct prompt/tool/model. Adopt a workflow framework as soon as you need any of: cycles/retries, persistence across a pause (human approval), or reliable observability into a multi-step process — hand-rolling these yourself is where most homegrown agent systems accumulate bugs.

---

**Q65: Token optimization — practical techniques to reduce LLM costs, and how usage/cost is actually measured.**

- **Context compression** — summarize or trim retrieved chunks/conversation history before sending to the LLM, rather than sending everything verbatim.
- **Semantic retrieval** (vs. sending full documents) — retrieve only the most relevant chunks (via embedding similarity) instead of stuffing entire documents into the prompt.
- **Chunk filtering** — after retrieval, filter out low-relevance chunks (via a relevance score threshold) before they ever reach the LLM, rather than relying on the model to ignore irrelevant context.
- **Prompt caching** — cache the static portion of a prompt (e.g., a long system prompt) so the provider doesn't re-process/re-charge full price for those tokens on every call (several providers offer discounted "cached" input tokens for exactly this).
- **Query rewriting** — rewrite a verbose or poorly-phrased user query into a more efficient, precise query before retrieval, reducing wasted/irrelevant retrieval.
- **Dynamic context loading** — only load additional context (e.g., a user's full history) when the specific request actually needs it, rather than always attaching maximum context to every call.
- **Model routing** — send simple queries to a cheap/fast model, and reserve the most expensive model for requests that genuinely need its capability.
- **Output control** — set `max_tokens` deliberately rather than leaving it unbounded, especially for structured/short-answer tasks.
- **Semantic caching** — serve a cached response for duplicate/near-duplicate queries, skipping the LLM call entirely.

**How this is measured in practice:** track **tokens per request** (input and output tracked separately, since providers often price them differently), **cost per request/session**, **cache hit rate** (how much traffic is being absorbed by caching instead of hitting the LLM), and **cost per successful outcome** (not just cost per call — a cheap call that fails and needs a retry may cost more overall than a slightly pricier call that succeeds the first time).

---

**Q66: How do you safely execute AI-generated outputs? Discuss schema validation, allow-lists, output parsing, approval workflows, confidence thresholds, and retry mechanisms — with an emphasis on production safety.**

- **Schema validation** — never execute a raw, unvalidated model output. Require structured output (e.g., a Pydantic model in Python) and validate it against a strict schema before acting on it; reject/retry on validation failure rather than attempting to "clean up" malformed output.
- **Allow-lists** — the agent should only ever be able to invoke a pre-approved, explicit set of tools/actions — never dynamically construct and execute arbitrary code or commands the model generated freely.
- **Output parsing** — parse structured output deterministically (JSON schema, function-calling structured returns) rather than trying to regex/interpret free-form text, which is fragile and a source of silent bugs.
- **Approval workflows** — gate any action with real-world consequences (financial transactions, data deletion, external communication) behind explicit human approval, especially early in a system's life before its reliability is well-established.
- **Confidence thresholds** — if the model/tooling can produce a confidence or relevance score, use it: high confidence → proceed automatically; low confidence → escalate to a human or a fallback path, rather than proceeding blindly.
- **Retry mechanisms** — validation failures should trigger a bounded retry (e.g., re-prompt with the specific validation error so the model can correct itself), capped at a small number of attempts before escalating to a human or failing gracefully — never an unbounded retry loop.
- **Production safety framing:** treat every AI-generated output as **untrusted input** until it passes validation — the same posture you'd take toward user input in a traditional web application. The goal is a system that fails safely and visibly (a rejected/escalated action) rather than one that executes something wrong silently.

---

**Q67: How do you prevent hallucination? Discuss RAG grounding, citations, reranking, response validation, confidence scoring, and tool-based verification.**

- **RAG grounding** — ground responses in retrieved, real source material, and explicitly instruct the model (in the system prompt) to answer only from that material, saying "I don't know" rather than guessing when the context doesn't contain the answer.
- **Citations** — require the model to cite which specific source/chunk supports each claim; this both discourages fabrication (the model has to point to something real) and makes unsupported claims easier to catch, by a human or an automated checker.
- **Reranking** — improves the *quality* of what's grounding the answer in the first place; a cross-encoder reranking pass surfaces the genuinely most relevant chunks, reducing the chance the model reaches for its own training knowledge because the retrieved context wasn't actually useful.
- **Response validation** — an automated faithfulness check (e.g., a RAGAS-style metric, or a second LLM call) that verifies each claim in the response is actually supported by the retrieved context, flagging or blocking responses that aren't.
- **Confidence scoring** — where available, use retrieval relevance scores and/or model-reported confidence to gate the response — low-confidence answers can be routed to a fallback (broader search, human handoff) instead of being shown as if authoritative.
- **Tool-based verification** — for factual/numeric claims where an authoritative source exists (a database, a calculator, an API), have the agent verify the claim against that source directly rather than trusting the LLM's internal "recollection" of a number or fact.

---

**Q68: AI evaluation — explain offline and online evaluation strategies, key metrics, and evaluation frameworks (LangSmith, DeepEval, Promptfoo, Azure AI Evaluations, OpenAI Evals).**

**Offline evaluation** — run before deployment, against a fixed dataset:
- A **golden dataset** of representative queries with expert-verified correct answers, including edge cases and adversarial/trick queries.
- Automated metrics: faithfulness, answer relevance, context precision/recall (RAGAS-style, for RAG systems); trajectory/tool-accuracy (for agents); code-execution tests (for code generation).
- **LLM-as-judge** scoring for open-ended quality dimensions (helpfulness, tone, groundedness), calibrated periodically against human ratings.
- Used as a **gate** before deployment — e.g., in CI/CD, block a release if metrics regress beyond a threshold.

**Online evaluation** — measured in production, on real traffic:
- User feedback signals (thumbs up/down, explicit ratings).
- Real-time faithfulness/safety checks on live responses (sampling, not necessarily every single response, for cost reasons).
- Business metrics — task completion rate, escalation rate, user retention/satisfaction.
- Drift detection — is quality degrading over time (e.g., due to a silent underlying model update, or a shift in real user query patterns away from what the golden set covers)?

**Key metrics across both:** faithfulness, answer relevance, context precision/recall, hallucination rate, latency, cost per request, and — for agents specifically — trajectory correctness and tool-call accuracy (did it take the right *path*, not just land on a plausible final answer).

**Frameworks (current landscape):**
- **LangSmith** — tracing, debugging, and evaluation tightly integrated with LangChain/LangGraph applications.
- **DeepEval** — open-source, **pytest-native** evaluation framework; strong built-in metric library (including "G-Eval," which lets you define a custom evaluation criterion in plain English and have it scored via an LLM-judge rubric); integrates cleanly into CI pipelines (Jenkins, GitHub Actions, Azure DevOps).
- **Promptfoo** — open-source, **YAML-config-driven** tool for prompt/model comparison testing and especially strong for **red-teaming/security testing** (built-in adversarial test generation for prompt injection, jailbreaks, PII leakage). Worth noting: OpenAI agreed to acquire Promptfoo in 2026, planning to fold it into OpenAI Frontier — it remains open source (MIT license) for now, but this is worth being aware of if vendor-neutral tooling matters for your evaluation stack.
- **Azure AI Evaluations** — Microsoft's native evaluation tooling within Azure AI Foundry, with built-in metrics (groundedness, relevance, coherence, safety) that integrate directly with Azure OpenAI-based applications.
- **OpenAI Evals** — a lightweight, template-based evaluation harness (including model-graded/LLM-judge evaluation), useful for quick, reusable task-specific test definitions.

**Practical takeaway:** no single tool covers everything — many production teams layer a pytest-style framework (DeepEval) for CI gating, a tracing platform (LangSmith or similar) for production observability, and a dedicated red-teaming pass (Promptfoo-style) for security before launch.

---

**Q69: How do you identify scenarios requiring manual approval, and explain confidence-based escalation and workflow resumption.**

**Identifying approval-required scenarios:**
- **Irreversibility** — can this action be undone if wrong? (sending money, sending an external email, deleting data → yes, requires approval; reading data or drafting a response for review → generally lower risk.)
- **Blast radius** — does this affect one user, or many (e.g., a bulk operation)?
- **Regulatory/compliance sensitivity** — actions touching financial, medical, or legal domains typically warrant approval regardless of the agent's confidence.
- **Novel/out-of-distribution requests** — if the request doesn't closely resemble anything in the golden/training examples, treat it as higher risk by default.

**Confidence-based escalation:**
- Attach a confidence signal to the decision point — this can come from retrieval relevance scores, the model's own expressed certainty, or agreement across multiple independent evaluations of the same output.
- Define explicit thresholds: **high confidence + low risk** → proceed automatically; **low confidence OR high risk** → route to a human for approval; **very low confidence** → don't even present a "best guess," explicitly say the system can't determine an answer and hand off.
- Log every escalation decision (and its confidence score) so thresholds can be tuned over time based on real outcomes, rather than being a one-time guess.

**Workflow resumption:**
- The escalation point must be a **checkpoint**, not just a pause in a live process — the full state (what's been done, what's pending, the exact context needed to make the decision) is persisted externally.
- Once a human responds (approve/reject/modify), the workflow resumes exactly from that checkpoint — it should not need to re-run earlier steps, and should be able to survive the human taking minutes, hours, or days to respond.

---

**Q70: Design a complete agent workflow with multiple tool integrations — explaining retries, timeouts, audit logging, graceful degradation, and handling partial failures.**

**Scenario:** an agent that needs to call 3 tools (e.g., inventory API, pricing API, shipping API) to fulfill one user request.

1. **Per-tool timeout** — each tool call has its own timeout (e.g., 10–30 seconds); a hung external API can't stall the whole workflow indefinitely.
2. **Retry policy per tool** — transient failures (timeout, 5xx, rate limit) get retried with exponential backoff, capped at a small number of attempts (e.g., 3); non-transient failures (4xx, malformed input) fail immediately without wasting retries.
3. **Audit logging** — every tool call (inputs, outputs, latency, success/failure, retry count) is logged with a correlation/trace ID tying it back to the originating user request — essential both for debugging and for compliance/accountability in production.
4. **Partial failure handling** — if the pricing API fails but inventory and shipping succeeded, the workflow shouldn't necessarily fail the entire request: decide per-scenario whether a partial result is still useful (e.g., "here's availability and shipping estimate; pricing is temporarily unavailable") or whether the missing piece is load-bearing enough that the whole request must fail.
5. **Graceful degradation** — if a tool is down entirely (its circuit breaker has tripped after repeated failures), fall back to a lower-fidelity alternative if one exists (a cached last-known value, a simpler heuristic, or a "please try again shortly" response) rather than hard-failing every request that touches that tool.
6. **Circuit breaker** — after a threshold of consecutive failures for a given tool, stop calling it for a cooldown period entirely, rather than continuing to hammer a service that's clearly down.
7. **Final response assembly** — the agent composes its answer from whatever succeeded, being explicit with the user about anything that's degraded or unavailable, rather than silently presenting incomplete results as if complete.

---

**Q71: Production readiness — observability, monitoring, logging, tracing, security, scalability, and reliability. What does *real* production experience look like, vs. a proof-of-concept?**

A proof-of-concept typically stops at "it works on my test queries." Production readiness means:

- **Observability & tracing** — every request traceable end-to-end (which tools were called, what the retrieval returned, what the model generated, at every step) — not just a final input/output log. Tools like LangSmith, Azure Application Insights, or an OpenTelemetry-based setup are used specifically so a failure can be diagnosed after the fact, not just noticed.
- **Monitoring & alerting** — dashboards for latency (p50/p95/p99, not just averages), error rate by category, cost per request, and business metrics (task success rate) — with alerts firing on anomalies, not just periodic manual review.
- **Structured logging** — every log entry includes timestamp, request/trace ID, user ID, tokens used, latency, and success/failure — searchable and correlatable, with **PII scrubbed** before it's written.
- **Security** — input guardrails (prompt injection detection, PII masking), output guardrails (leak detection), access control (auth + role-based filtering on retrieved data), and secrets management (never hardcoded, always via a vault).
- **Scalability** — the system is load-tested (not just "seems fine at low volume"), with auto-scaling, rate limiting, and caching validated under realistic concurrent load, not assumed.
- **Reliability** — provider failover, circuit breakers, graceful degradation, and a tested rollback plan — validated by actually simulating failures (e.g., killing a dependency in staging) rather than just designing for it on paper.
- **Evaluation as an ongoing process, not a one-time gate** — a living golden dataset that grows with every real production failure, regression checks on every deployment, and canary rollouts — not a single evaluation run before the initial launch and then never revisited.

**The core distinguishing mindset:** a PoC is built to demonstrate the happy path works. Production readiness is built around the assumption that things *will* fail — tools will time out, models will occasionally hallucinate, traffic will spike unpredictably — and the system is designed so those failures are contained, visible, and recoverable rather than surprising.

---

# Part 4: Testing, Evaluation & Monitoring

## Testing AI Systems

**Q1: We're building a code generator — how do you test that it's working correctly?**

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

## Monitoring & Reliability

**Q2: How do you monitor a Gen AI app in production?**

- **Quality metrics** — faithfulness, relevance, hallucination rate
- **Performance** — latency
- **Cost** — tokens per request
- **User feedback**
- **Alerting**
- **System health** — API errors, timeouts, rate limits

**Q3: Your Gen AI app worked well for 6 months, then started failing — why?**

- OpenAI/provider silently updated the underlying model — re-tune your prompt.
- Query shift — real user queries have drifted from what was originally tested.
- Vector DB pollution.
- A LangChain (or other framework) version update changed behavior.

**Q4: How do you A/B test prompts?**

Run two or more prompt variants against the same input set and compare results.

**Q5: How do you build a regression test suite?**

- Collect 100–500 real user queries from production representing edge cases.
- Evaluate against them.
- Halt deployment if any critical metric drops by more than 5%.

---

---

# Part 5: Safety, Security & Compliance

## Safety, Security & Compliance

**Q1: How do you build a secure Gen AI app for a hospital?**

- Use a secure, compliant LLM provider.
- De-identify data before sending it to the LLM — remove names, IDs, addresses.
- Encrypt everything in transit (TLS) and at rest (AES-256).
- Implement logging.

**Q2: How would you build a system for a hospital to summarize patient notes for doctors?**

- Strip patient names/IDs from notes.
- Send anonymized text to Azure OpenAI (or similar) to get a summary.
- Re-insert the patient's name only for the authorized doctor.

**Q3: How do you prevent sensitive data from leaking through LLM responses?**

- **Input filtering** — scan user queries to catch sensitive data.
- **Context filtering** — flag sensitive data like account numbers, IDs, etc.
- **Output filtering** — check responses with regex + ML before returning them to the user.
- Log every interaction for later review.
- Use guardrail tools like Microsoft Presidio.

**Q4: How do you implement role-based access control (RBAC) in RAG?**

- When indexing, attach metadata to each document (user role, department, access level).
- When a user queries, extract claims from their JWT token.
- Filter vector search results by access level.
- Add a final access check in the LLM prompt itself.
- Always pull the user's role from the JWT token.

**Q5: How do you handle PII?**

- Scan all training data using tools like Microsoft Presidio.
- Mask PII with placeholders.
- Manually review and log all transactions.

**Q6: How do you defend an LLM app against prompt injection?**

- **Input wrapping** — enclose user input in delimiters (e.g., XML tags) and instruct the LLM to treat it as data, not instructions.
- Use a separate LLM to detect injection attempts.
- Don't give the LLM access to sensitive tools without user confirmation.
- Monitor outputs.

**Q7: How do you handle an offensive response if one occurs?**

- Disable the feature that produced it.
- Post a public acknowledgment.
- Trace the root cause.
- Implement guardrails.
- Add the offensive response to the test suite.

---

---

# Part 6: Azure & Microsoft Ecosystem

## Microsoft Copilot Studio / Azure AI Foundry

**Q1: What is Azure AI Foundry agents?**

Supports different agent types, including **declarative agents** (configuration-driven, no custom code) and **hosted agents** (custom code hosted and orchestrated by the platform).

**Q2: What is Microsoft 365 Copilot / Work IQ integration?**

Connects an AI assistant to your Microsoft 365 (Copilot) data so the agent can query workplace information — documents, emails, meetings, etc. — using natural language.

**Q3: When should you use Copilot Studio?**

- Built around **topics** — a topic represents a subject/intent the user is asking about within a conversation.
- Can integrate with API connectors and automation (e.g., Power Automate).
- Well suited for department- or organization-specific agents, and customer-facing agents.
- Supports deployment to Microsoft Teams and other channels.

**Q4: What is a system topic in Copilot Studio?**

Pre-built topics that handle scenarios almost every agent needs — greetings, escalation to a human, handling multiple topics matching a query, fallback/"I didn't understand," etc. — so you don't have to author them yourself.

**Q5: What is quota (in Copilot Studio)?**

Limits on how many messages/sessions the agent can handle in a given period — tied to licensing, not just a technical rate limit.

**Q6: What is generative AI orchestration?**

The agent answers user queries or reacts to event triggers by dynamically selecting the most appropriate combination of **topics, tools, and knowledge sources** to handle the request — as opposed to following one fixed, hardcoded path.

**Q7: What is classic orchestration?**

Each topic has a defined set of **trigger phrases**; the agent uses NLP to match the user's input to the closest topic. More rule-based/deterministic than generative orchestration.

**Q8: How would you design a Microsoft Teams channel with agentic AI?**

*(Azure AI Foundry was rebranded to **Microsoft Foundry**, and the Teams integration path has evolved.)*

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

**Q9: What is Azure AI Search?**

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

## Azure AI Content Safety / Content Filters

**Q10: What is the content filter in Azure AI Foundry?**

A safety layer that operates on both sides of a model call:
- **Input filtering** — blocks malicious/harmful user input before it ever reaches the model.
- **Output filtering** — monitors what the model generates and blocks harmful, inaccurate, or copyright-infringing content before it's returned to the user.

**Q11: What does Azure AI's content filter actually check for?**

- **Four core harm categories**: hate/fairness, violence, self-harm, sexual content — each scored with a severity level.
- **Prompt Shields** — detects both **direct** prompt attacks (jailbreak attempts embedded in the user's own message) and **indirect** attacks (malicious instructions hidden inside retrieved documents/content the model is asked to process).
- **Protected material detection** — checks model-generated text/code against known public repositories and copyrighted text to catch verbatim reproduction.
- **Custom blocklists** — you can define your own list of terms to explicitly block, layered on top of the built-in categories.

**Q12: What is quota in Azure AI Foundry / Azure OpenAI?**

Assigned at the **subscription level, per region, per model** — expressed as:
- **TPM** — tokens per minute
- **RPM** — requests per minute

**Global deployment** types use Azure's global infrastructure to dynamically route traffic to whichever datacenter has capacity available, rather than pinning you to one region — useful for higher throughput/availability at the cost of not controlling exactly which region processes a given request.

---

## Azure Identity

**Q13: What is Azure Managed Identity?**

*(Clarified: this describes system-assigned managed identity specifically — the next entry covers the user-assigned variant.)* A passwordless authentication feature in Azure where an Azure resource is automatically given its own identity, letting it securely connect to other Azure resources without a developer ever having to manage or store a secret/credential themselves.

**Q14: What is a user-assigned managed identity?**

A standalone Azure resource, created independently of any specific service, that represents an identity you can attach to multiple Azure services at once — letting several resources share the same identity and permission set, rather than each getting its own separate system-assigned identity.

---

## Azure AI Search — Deep Dive

**Q15: How does Azure AI Search work, at a high level?**

1. **Data source** — connects to where your raw content lives (Blob Storage, SQL, SharePoint, Cosmos DB, etc.).
2. **Indexer** — pulls content from the source, optionally runs it through a **skillset** (chunking, OCR, entity extraction, embedding generation), and writes the results into a **search index**.
3. **Search index** — the structured, queryable store — with searchable text fields (for keyword/BM25 search) and vector fields (for semantic/vector search).
4. **Querying** — supports keyword search, pure vector search, or **hybrid search** (both combined via Reciprocal Rank Fusion), optionally followed by a **semantic ranker** re-ranking pass for improved relevance.

**Q16: A user is searching SharePoint (containing documents, JPEG images, and vector images/blueprints) for "a computer blueprint" that exists somewhere in the file system or SharePoint. How would Azure AI Search retrieve that image?**

This is a **multimodal search** scenario, and Azure AI Search has built-in support for it:

1. **Ingestion:** a SharePoint indexer (or a Blob Storage indexer, if content is synced there first) pulls in the documents and images during indexing.
2. **Image understanding**, via one of two approaches:
   - **Image verbalization** — during ingestion, a GenAI/LLM skill generates a natural-language description of the image (e.g., "computer motherboard blueprint showing CPU socket, RAM slots, and power connectors"). That description is stored as text and embedded, so it participates in both keyword *and* vector search just like any other text content.
   - **Direct multimodal embeddings** — a multimodal embedding model (e.g., Azure AI Vision's multimodal embedding model) embeds the image directly into the **same vector space** as text queries, without needing a text description step. A text query like "computer blueprint" gets embedded and compared directly against the image's embedding via vector similarity.
3. **Querying:** when the user searches "computer blueprint," Azure AI Search runs a **hybrid query** — keyword matching against any verbalized descriptions/surrounding text, plus vector similarity matching against image embeddings (whichever approach was used) — merges the results, and optionally reranks them.
4. **Result:** the matching image (or the document/page it came from) is returned, along with a reference/path back to where it physically lives (SharePoint or the underlying file system), so the user can open the original file.

This is exactly the kind of scenario Azure AI Search's multimodal search capability (built-in skills for extracting, describing, and embedding both text and images from documents like PDFs) is designed to solve.

**Q17: Are you familiar with Microsoft Copilot Studio?**

*(Model answer — personalize with your own project details.)* Copilot Studio is a low-code platform for building conversational agents ("Copilot" bots), built around **topics** (conversation subjects/intents), with support for API connectors, Power Automate integration, and deployment to channels like Teams and websites. It's well suited to department-specific or customer-facing agents that don't need heavy custom code. If asked "have you used it," describe: what kind of agent you built with it, which topics/system topics you configured, how you integrated external data (API connectors or generative orchestration pulling from a knowledge source), and how it was deployed (e.g., to Teams).

**Q18: How is a Copilot chatbot typically deployed?**

*(Model answer.)* Built and configured in Copilot Studio (or Microsoft Foundry Agent Service for a code-first agent) → published to one or more channels (Microsoft Teams, a custom website via embed, Microsoft 365 Copilot) → for Teams specifically, publishing can provision the required Azure Bot Service registration automatically → ongoing monitoring via the platform's built-in analytics or Azure Application Insights.

---

---

# Part 7: Production Deployment & Cost

## Cost & Performance

**Q1: Your LLM bill hit ₹50 lakh/month — how do you cut it by 70%?**

- **Model routing** — ~70% of queries don't need a high-reasoning model like GPT-5; route them to smaller models like GPT-mini.
- **Prompt caching**
- **Shorter prompts** — remove unnecessary examples.
- **Cache frequent queries.**

**Q2: Your chatbot needs to respond in under 500ms — what do you do?**

- **Streaming** — first token in 100–200ms feels instant.
- Use smaller models.
- Prompt caching.
- Put servers closer to users (edge deployment).
- Pre-compute embeddings for vector search.
- Reduce max tokens.

**Q3: How do you scale an application?**

- Load balancers
- Horizontal scaling — run multiple copies of the app
- Make everything async
- Caching
- Rate limiting

---

## Building & Deploying AI Projects

**Q4: How do you approach building an AI project from scratch?**

1. Define the problem clearly.
2. Understand the data — format (database, PDF, API), volume, and quality.
3. Build a simple prototype.
4. Evaluate — accuracy, precision/recall, user satisfaction.
5. Improve iteratively.
6. Deploy — with guardrails, security review, and monitoring in place.

**Q5: How do you choose the right LLM for a production use case?**

1. **Capability** — does the model actually handle the task? Test on your real use case, not a generic benchmark.
2. **Speed** (latency).
3. **Cost** — input vs. output token pricing (often priced differently).
4. **Context window** — how much text needs to fit in a single call.
5. **Privacy & compliance** requirements.
6. **Reliability** — what uptime does the provider guarantee, and what's the fallback plan if it's down?

**Q6: GPT vs. Claude vs. Gemini vs. Llama vs. Mistral — when to use each?**

*(Worth a caveat: model capabilities and relative strengths shift fast — verify against current benchmarks before quoting this in an interview rather than treating it as fixed.)*

- **GPT** — strong for agentic use, tool calling, structured data extraction.
- **Claude** — strong at long-document analysis, code generation, complex reasoning.
- **Gemini Flash** — fastest and cheapest for multimodal, real-time applications.
- **Llama** — free to use, customizable, can run on-premises.
- **Mistral** — strong at European languages, good cost-to-performance ratio.

**Q7: When should you use prompt engineering vs. RAG vs. fine-tuning?**

- **Prompt engineering** — when the task is solvable with good instructions and under ~500 examples.
- **RAG** — when you need internal/company data, recent events, or knowledge that changes frequently and must stay up to date.
- **Fine-tuning** — when you need a specific output format/style that prompting can't achieve consistently, and you have 1,000+ examples.

---

## Designing an LLM/Gen AI Product — Key Considerations

**Q8: What are the key challenges/considerations when designing an LLM-based product?**

- **Requirement gathering** — what specific problem is this actually solving?
- **Edge case identification** — what happens when the user asks something outside the intended scope?
- **Legal/policy constraints** — are there compliance, IP, or regulatory considerations?
- **Expected business outcomes** — defined success metrics, acceptable accuracy thresholds.
- **Data source & citation** — where does grounding data come from, and can answers be traced back to sources?
- **Integration points** — internal systems, third-party systems, and how authentication is handled across them.
- **Governance & maintenance** — audit trails, feedback loops, and ongoing data security considerations.

---

## Tools & Frameworks

**Q9: What is Pydantic?**

A Python library for **data validation and settings management using type hints**. You define a model as a class with typed fields; Pydantic validates incoming data against those types at runtime, coerces compatible types, and raises clear validation errors otherwise. In AI/agent systems specifically, it's commonly used to force an LLM's output into a **structured, guaranteed-shape response** (e.g., "return a `PatientSummary` object with these exact fields") rather than trusting free-form text.

**Q10: What is LangGraph?**

A framework (from the LangChain team) for building **stateful, multi-step AI workflows and multi-agent systems as a graph** of nodes and edges, rather than a simple linear chain. Key capabilities that distinguish it from a basic LangChain chain:
- Supports **cycles/loops** (e.g., an agent retrying or re-planning), not just straight-line execution.
- Built-in **state persistence** across steps, so long-running or resumable workflows are possible.
- Native support for **human-in-the-loop** checkpoints (pause and wait for approval before continuing).
- Used for orchestrating **multi-agent** systems where different nodes represent different agents/tools collaborating.

---

## Production Experience — Deployment, APIs & Scaling

*(The following are experience-based interview questions. Below are model answer frameworks — the technical substance a strong answer should cover — for you to personalize with your own project specifics.)*

**Q11: What experience do you have with deployment and production rollout?**

A strong answer covers: containerizing the application (Docker), deploying to a managed compute service (Azure App Service, AKS, or a container app), using **feature flags** to roll out new capability safely, **health checks** for orchestration platforms to detect and restart unhealthy instances, and **automated rollback** if a deployment's error rate spikes. For AI-specific rollouts specifically, mention **canary deployment** — routing a small percentage of traffic (e.g., 5%) to a new model/prompt version before a full rollout, monitored against your evaluation metrics.

**Q12: Have you implemented REST APIs for AI model integration? Did you develop REST APIs in Python?**

A strong answer names the concrete stack — most commonly **FastAPI** in Python for exposing an LLM/agent as a REST endpoint, with request/response schemas defined via **Pydantic models** for validation, and describes a real endpoint (e.g., `/chat` accepting a user query + session ID, returning a streamed or complete response).

**Q13: In FastAPI, how do you typically design APIs, and how do you handle async operations and performance?**

- Define request/response schemas explicitly with **Pydantic models** — this gives you automatic validation and OpenAPI docs for free.
- Use `async def` route handlers for I/O-bound work (calling an LLM API, a database, another service) so FastAPI can handle other requests concurrently while waiting — this is where async actually pays off, as covered in the .NET async section's underlying principle (don't block a thread on I/O).
- For **streaming** LLM responses, use `StreamingResponse` to stream tokens back to the client as they're generated, rather than waiting for the full response.
- For genuinely CPU-bound work, offload to a background worker/task queue rather than blocking the event loop.
- Add dependency injection (FastAPI's `Depends`) for things like auth, DB sessions, and shared clients, rather than instantiating them per request.

**Q14: Do you have experience with containerization? Are you aware of multi-stage builds in Docker?**

- **Containerization experience:** packaging an app (e.g., a FastAPI service) with its dependencies into a Docker image, so it runs identically across dev/staging/production, and deploying that image to a container platform.
- **Multi-stage builds:** using multiple `FROM` stages in a single `Dockerfile` — e.g., one stage that installs build tools and compiles/installs dependencies, and a second, minimal final stage that copies over only the built artifacts and runtime dependencies (not the build tools). This keeps the final image significantly smaller and reduces the attack surface, since compilers and build-time-only packages never end up in the shipped image.

```dockerfile
# Stage 1: build
FROM python:3.12 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: final, minimal runtime image
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]
```

**Q15: If your AI API suddenly gets high traffic, what steps would you take to scale it? How would you scale to handle 10,000 concurrent requests?**

1. **Horizontal scaling / auto-scaling** — add more instances behind a load balancer, scaling out based on CPU/request-queue metrics.
2. **Async, non-blocking request handling** — ensure the API isn't blocking threads on I/O (LLM calls, DB calls), so each instance can handle many concurrent in-flight requests.
3. **Semantic caching** — absorb a meaningful chunk of duplicate/near-duplicate traffic without hitting the LLM at all.
4. **Model routing** — route simpler requests to cheaper/faster models, reserving the most expensive model for requests that truly need it.
5. **Rate limiting + queuing** — protect the system (and the downstream LLM provider's own rate limits) from being overwhelmed; queue excess load rather than dropping it outright.
6. **Circuit breakers** — if the LLM provider itself is struggling, fail fast and fall back (a secondary provider, a cached response, or a graceful "try again shortly" message) instead of piling up hung requests.
7. **Multi-region deployment** — for both latency and resilience, once traffic is large enough to justify it.

**Q16: Have you worked with Azure OpenAI? What configurations did you change, or what Azure services did you work with, and how did you improve search strategy? What were end users trying to search, and what's the criteria for a successful query?**

*(Model answer framework.)* A strong answer names: the specific Azure OpenAI deployment(s) used (model, region, quota/TPM-RPM settings tuned for the load), and — for search-strategy specifically — concrete tuning decisions such as: adjusting chunk size/overlap after seeing retrieval misses, switching from pure vector to **hybrid search** once exact-term queries (product codes, names) were being missed, adding the **semantic ranker** for a relevance boost, or adjusting the number of retrieved chunks (`top-k`) based on context window budget. For "criteria for a successful query," a good answer defines it concretely: the correct document/chunk appears in the top-k results (retrieval quality), the generated answer is faithful to that content (no hallucination), and — ideally — this is validated against a labeled/golden query set rather than judged anecdotally.

**Q17: When doing prompt engineering, how do you ensure responses come out correctly, and what challenges have you faced while tuning prompts?**

- Iterate against a fixed **test set of representative queries** (including edge cases) rather than eyeballing a handful of examples.
- Use explicit **structure** in the prompt — clear instructions, examples (few-shot), and a defined output format (e.g., requesting JSON with a schema) to reduce variance.
- **Common challenges worth naming:** the model ignoring instructions once the prompt gets too long; inconsistent output format across runs (mitigated with structured output / Pydantic-validated JSON); the model answering from its own training knowledge instead of provided context (mitigated with explicit grounding instructions and a faithfulness check); and prompts that worked well on one model version behaving differently after a silent model update — which is why a regression test suite matters.

**Q18: Can you explain the difference between Azure Functions and Durable Functions? Where have you used each?**

- **Azure Functions** — stateless, event-driven, short-lived function executions triggered by events (HTTP request, timer, queue message, blob upload). Good fit for simple, quick, independent tasks (e.g., "resize this uploaded image," "process this single API call").
- **Durable Functions** — an extension of Azure Functions that adds **statefulness and orchestration**, letting you write long-running, multi-step workflows in code (rather than a low-code designer), with automatic checkpointing so a workflow can survive restarts and run for hours, days, or longer. Supports patterns like function chaining, fan-out/fan-in (parallelize then aggregate), async HTTP APIs (long-running operation + status polling), and human-interaction/waiting patterns.
- **When to use which:** a single, quick, independent task → plain Azure Function. A multi-step process that needs to track state across steps, wait on external events, or fan out work and later combine results → Durable Functions.

**Q19: How are you monitoring your applications in production? Have you used Azure Application Insights, and how have you improved based on it?**

*(Model answer framework.)* Application Insights (part of Azure Monitor) provides distributed tracing, request/dependency telemetry, custom metrics, and log queries (via Kusto/KQL). A strong answer describes: tracking custom metrics specific to an AI application (LLM latency, token usage/cost per request, retrieval quality signals, error rates by category), setting up alerts on anomalies (e.g., latency spikes, error rate crossing a threshold), and giving a concrete example of a change made *because of* something observed in the telemetry (e.g., "we noticed P95 latency spiking during a specific retrieval step and added caching there").

---

---

# Part 8: Python

## Python / Asyncio

- The **event loop** is the central hub that schedules and runs coroutines.
- Calling an `async def` function doesn't execute it immediately — it returns a **coroutine object**.
- A coroutine object only starts running once it's **awaited** (`await`) or scheduled as a task.
- `asyncio.run()` starts the event loop and runs a top-level coroutine to completion.
- A **Task** (`asyncio.create_task()`) wraps a coroutine and schedules it on the event loop, allowing multiple coroutines to run *concurrently* (interleaved, not in parallel).
- CPython's **GIL (Global Interpreter Lock)** allows only one thread to execute Python bytecode at a time — this is why `asyncio` uses a single-threaded event loop for concurrency rather than true multi-threaded parallelism. It's best suited for I/O-bound work (network calls, file I/O), not CPU-bound work.

---

## Python

### Modules & Packages

**Q1: What is a module in Python?**

A single `.py` file that groups related code together — functions, classes, and variables — that can be imported and reused elsewhere.

**Q2: What is a package?**

A directory containing multiple modules, letting you group related modules together under a single namespace.

---

### Core Language Basics

**Q3: What is pandas?**

A library used for data manipulation and analysis — built around the `DataFrame`, a table-like structure for working with structured/tabular data.

**Q4: What types of loops does Python have?**

- `for` loop
- `while` loop
- **List comprehension** — a compact, "one-line loop" way to build a list.

```python
squares = [x * x for x in range(10)]
```

**Q5: `map` vs. `filter`?**

- **`map(function, iterable)`** — applies `function` to every item in `iterable`, returning an iterator of the results.
- **`filter(function, iterable)`** — keeps only the items where `function(item)` returns `True`, discarding the rest.

**Q6: How does Python handle indentation?**

Python uses whitespace (indentation) to define code blocks, instead of curly braces `{}` like many other languages.

**Q7: What is `self`?**

Refers to the current instance of a class — used inside instance methods to access that specific object's variables and methods.

**Q8: What is a tuple?**

An ordered, **immutable** sequence — once created, it cannot be changed. Defined using parentheses, e.g. `(1, 2, 3)`.

**Q9: How does slicing work?**

Extracts a specific portion of a sequence (string, list, tuple, etc.) using `sequence[start:stop:step]`.

- `sequence[::2]` — every 2nd element.
- `data[-3:]` — the last 3 elements (negative indices count backward from the end).
- `name[::-1]` — reverses the sequence.

**Q10: How is a Python list actually stored in memory?**

A Python list is implemented as a **dynamic array of references (pointers)** — that array of pointers *is* stored in one contiguous block of memory, but each pointer just points to wherever the actual object happens to live in memory (objects themselves are scattered around the heap, not stored inline in the list). This is different from, say, a C array of raw integers, where the actual values sit contiguously — a Python list's contiguous block holds *addresses*, not the values themselves.

**Q11: Class basics?**

- `__init__` — the constructor, called automatically when an instance is created.
- `self` is always the first parameter of any instance method (Python passes it automatically).
- To create an instance, call the class like a function: `obj = MyClass(...)`.

**Q12: What is a coroutine?**

A coroutine is a specialized function that can **pause its execution** and **resume later**, letting other code run in the meantime — the basis of Python's `async`/`await` model (cooperative, single-threaded concurrency).

This is **not** the same thing as the GIL. The **GIL (Global Interpreter Lock)** is a separate mechanism in CPython that ensures only one thread executes Python bytecode at a time, regardless of whether coroutines are involved — it's about *threading*, not about coroutines. Coroutines achieve concurrency *without* needing multiple threads at all — a single thread cooperatively switches between coroutines at `await` points.

**Q13: What is a lambda function?**

An anonymous (unnamed) function, typically for short, throwaway logic:

```python
f = lambda a: a * a
```

**Q14: What is a decorator?**

A function that wraps another function to add extra behavior, without modifying the original function's code:

```python
def my_decorator(func):
    def wrapper():
        print("before execution")
        func()
        print("after execution")
    return wrapper

@my_decorator
def greet():
    print("Hello world!")
```

---

### Environment & Dependency Management

**Q15: What is a virtual environment?**

A self-contained directory for installing packages specific to one project, isolated from the system-wide Python installation — keeps your global Python environment clean and prevents dependency conflicts between projects.

**Q16: What is `requirements.txt`?**

A file listing the libraries (and often their versions) a project depends on.

```bash
python -m venv .venv

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# macOS/Linux
source .venv/bin/activate
```

Example `requirements.txt`:

```text
langchain
openai
fastapi
uvicorn
```

```bash
# Install packages from the file
pip install -r requirements.txt

# Generate the file from what's currently installed
pip freeze > requirements.txt
```

**Q17: What is a lock file?**

Records the **exact resolved version** of every package (and its dependencies) actually installed — used to reproduce an identical environment elsewhere, rather than just "install whatever's compatible right now" as a loose `requirements.txt` might.

Common examples: `uv.lock`, `poetry.lock`, `Pipfile.lock`.

---

### Data Structures

**Q18: What are Python's core built-in data structures?**

- **List** — ordered, mutable, allows duplicate values.
- **Tuple** — ordered, immutable; generally faster than a list for fixed data, and — because it's immutable (hashable) — can be used as a dictionary key, unlike a list.
- **Set** — unordered, no duplicates, extremely fast membership lookup (`in` checks).
- **Dictionary** — key-value pairs, fast lookup by key.

**Q19: Ordered vs. unordered data structures?**

- **Ordered** — preserves the order elements were inserted in; you can rely on iterating/accessing them in that order.
- **Unordered** — doesn't guarantee any particular order; often used specifically when you need fast lookups or uniqueness rather than order (e.g., a `set`).

*(Worth knowing for an interview: since Python 3.7, the built-in `dict` officially guarantees insertion order as a language feature — so in modern Python, `dict` is technically both "key-value" and "ordered," which surprises people expecting dictionaries to be inherently unordered.)*

**Q20: What are generators in Python?**

A generator is a function that produces a sequence of values **lazily**, one at a time, instead of computing and returning them all at once. It uses `yield` instead of `return` — each call to `next()` on the generator resumes execution right where it left off, runs until the next `yield`, and pauses again.

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1

for num in count_up_to(5):
    print(num)  # 1, 2, 3, 4, 5 — computed one at a time, not all up front
```

Generators are memory-efficient for large or infinite sequences, since they never hold the entire sequence in memory at once — this is the same underlying mechanism behind `yield return` in C#, covered in the .NET notes.

**Q21: `*args` and `**kwargs`?**

- **`*args`** — collects any number of extra **positional** arguments into a tuple.
- **`**kwargs`** — collects any number of extra **keyword** arguments into a dictionary.

**Q22: What is `Literal` (from `typing`)?**

Restricts a value to one of a specific, fixed set of literal values — useful for catching typos or invalid values at type-check time rather than at runtime.

```python
from typing import Literal

def set_status(status: Literal["start", "end"]) -> None:
    ...
# set_status("startt")  # a type checker (e.g., mypy) would flag this as invalid
```

---

### More Python Fundamentals

**Q23: List vs. Tuple — key differences?**

| List | Tuple |
|---|---|
| Mutable | Immutable |
| Slower | Faster |
| More memory | Less memory |

The speed/memory advantage comes from immutability — since a tuple's size and contents can't change after creation, Python can allocate it more compactly and skip the bookkeeping needed to support resizing/mutation.

**Q24: Deep copy vs. shallow copy?**

```python
import copy

b = copy.copy(a)       # shallow copy
b = copy.deepcopy(a)   # deep copy
```

- **Shallow copy** — creates a new outer object, but nested objects inside it (e.g., a list of lists) are still shared references with the original. Modifying a nested object through `b` will also affect `a`.
- **Deep copy** — recursively copies every nested object too, so `b` is fully independent of `a` — modifying anything inside `b` never touches `a`.

**Q25: Process vs. Thread?**

- **Thread** — shares memory with other threads in the same process; lightweight to create; well suited to **I/O-bound** tasks (waiting on network/disk).
- **Process** — has its own separate memory space; heavier to create and communicate across (needs IPC); well suited to **CPU-bound** tasks, since separate processes can run on separate CPU cores in true parallel.

**Q26: What is the GIL in Python?**

The **Global Interpreter Lock** — a mechanism in CPython that allows only **one thread** to execute Python bytecode at a time, even on a multi-core machine.

- This is why **threads** are good for I/O-bound tasks in Python (they release the GIL while waiting on I/O, letting other threads run) but don't give you true CPU parallelism.
- For **CPU-bound** work, **multiprocessing** (separate processes, each with its own GIL) is generally preferred over threading, since it can actually use multiple cores.

**Q27: How is a Python dictionary implemented internally?**

Implemented using a **hash table**. A key's hash value determines where its entry is stored, giving:

- **O(1) average-case** lookup, insertion, and deletion.
- (Worst case degrades to O(n) if there are many hash collisions, though Python's hash table design makes this rare in practice for typical usage.)

**Q28: What's the difference between `__init__.py` and `__main__.py`?**

- **`__init__.py`** — tells Python that a directory should be treated as a **package**, and can contain initialization code that runs when the package is imported.
- **`__main__.py`** — defines what should happen when the **package itself** is executed directly (e.g., `python -m mypackage`), rather than when it's imported by other code.

---

# Part 9: Retrieval, Tokenization & Transformer Internals — Numbered Reference

**Q1: Why use token/fixed-size chunking over semantic chunking?**

Fixed-size (token-based) chunking is simpler, faster, and cheaper — it just counts tokens, with no embedding computation needed to decide split points. Semantic chunking requires computing and comparing embeddings between sentences across the whole document, which is meaningfully more expensive. Use token chunking when speed/cost matters more than perfect topical coherence, or when content is fairly uniform in structure. Use semantic chunking when retrieval quality on topically complex, unevenly-structured content is worth the extra compute cost.

**Q2: How do you scale LLMs (serving/inference)?**

- **Batching** — group multiple requests together for a single forward pass, improving GPU utilization.
- **Quantization** — run the model at lower numerical precision (e.g., INT8/FP8 instead of FP32) to cut memory and increase throughput, with a small accuracy tradeoff.
- **Model/tensor parallelism** — split a large model across multiple GPUs when it doesn't fit on one.
- **KV-cache optimization** — reuse cached key/value attention states across generation steps instead of recomputing them.
- **Horizontal scaling** — run multiple model replicas behind a load balancer.
- **Model routing, caching, and smaller distilled models** for requests that don't need the largest model — covered in more detail in the Cost & Performance section earlier in this doc.

**Q3: How do you observe an LLM app in production?**

Covered in depth in the Monitoring & Reliability section earlier in this doc (quality metrics, performance, cost, alerting, system health) — in short: trace every request, track faithfulness/latency/cost/error-rate metrics, and alert on anomalies rather than relying on manual review.

**Q4: Dense retrieval vs. sparse retrieval vs. hybrid retrieval — when would you use each?**

- **Dense retrieval (semantic search)** — uses neural embeddings (e.g., `text-embedding-3`, BERT-based models) that capture semantic meaning beyond exact keywords (e.g., "car" matches "automobile," "vehicle"). *Pros:* understands context, synonyms, paraphrasing. *Cons:* computationally more expensive, can miss exact-term matches. *Use when:* queries are natural-language and need semantic understanding.
- **Sparse retrieval** — traditional methods like TF-IDF and **BM25**; exact term matching. *Pros:* fast, interpretable, strong on specific terms. *Cons:* misses synonyms, no semantic understanding. *Use when:* queries contain specific codes, IDs, or exact phrases.
- **Hybrid retrieval** — combines both, typically via **Reciprocal Rank Fusion (RRF)**. Generally outperforms either alone in production, since real queries mix both needs. *Use when:* production systems need both accuracy and broad coverage.

Related techniques:
- **Multi-vector retrieval** (e.g., ColBERT) — each token in the document is embedded separately, and scoring happens via "late interaction" between query and document token vectors (see the ColBERT entry in Part 2).
- **Graph-based retrieval** — retrieval follows relationships (edges) between entities/documents represented as a graph (GraphRAG).
- **Reranking** — scores the initially retrieved candidates a second time for better precision.

**Q5: What is Reciprocal Rank Fusion (RRF)?**

A method for merging multiple ranked result lists (e.g., a sparse/BM25 ranking and a dense/vector ranking) into a single combined ranking, without needing the two lists' raw scores to be on comparable scales. For each document, its RRF score is:

`RRF_score(d) = Σ 1 / (k + rank_i(d))`

...summed across each ranking list `i` the document appears in, where `rank_i(d)` is that document's position in list `i`, and `k` is a small constant (commonly 60) that dampens the impact of very high ranks. Documents that rank well *across multiple* lists rise to the top of the fused ranking — this is exactly the mechanism hybrid search uses to combine BM25 and vector search results.

**Q6: Explain SPLADE. How does it compare to BM25?**

**SPLADE** (SParse Lexical AnD Expansion model) is a learned sparse retrieval method — it sits between pure BM25 and dense embeddings. Like BM25, it produces a **sparse vector** over the vocabulary (so it's still compatible with efficient inverted-index search infrastructure and remains interpretable — you can see which terms drove the match). Unlike BM25, the term weights (and even which terms are included) are learned by a transformer model, which also performs **term expansion** — adding semantically related terms the document doesn't literally contain (e.g., a document about "car" might get expansion weight on "vehicle" or "automobile"), something exact-match BM25 can never do. Net effect: SPLADE captures some of dense retrieval's semantic matching ability while keeping sparse retrieval's speed and interpretability advantages.

**Q7: Chunking vs. tokenization — how are they different?**

- **Tokenization** — breaking text into the smallest units for model input (characters, subwords, or words). Purpose: prepare text for the neural network to consume. Output: a list of tokens with IDs. Happens during model input preprocessing, on every single request.
- **Chunking** — breaking documents into larger, semantically meaningful segments (sentences, paragraphs, topic-based sections). Purpose: maintain enough context for retrieval and generation to work well. Output: a list of meaningful text segments. Happens during document *indexing*, before any queries even arrive — a one-time (or periodic) step, not a per-request one.

**Q8: How does tokenization affect the cost and performance of an LLM application?**

- **Cost is billed per token** (input and output, often at different rates), so how efficiently a tokenizer represents your text directly affects your bill — the same sentence can cost more or less depending on the tokenizer.
- **Non-English languages are often tokenized less efficiently** by tokenizers trained predominantly on English text, meaning the same sentence can take meaningfully more tokens (and cost more) in some languages than others.
- **The context window is measured in tokens**, not characters or words — inefficient tokenization eats into how much actual content fits in a single call.
- **More tokens = more compute = higher latency** — generation time scales with the number of tokens processed/generated.

**Q9: What is subword-level tokenization?**

Rare/uncommon words get split into smaller, meaningful subword pieces, while common words are kept whole — implemented via algorithms like **Byte Pair Encoding (BPE)** and **SentencePiece**. This balances vocabulary size against the ability to represent any word (even unseen ones) as a combination of known subword pieces. Examples: OpenAI's `tiktoken`, Google's SentencePiece.

**Q10: Compare fixed-size, recursive, and semantic chunking.**

- **Fixed-size chunking** — simple, predictable chunk sizes. *Con:* may split mid-sentence, breaking context.
- **Recursive chunking** — tries splitting on a prioritized list of separators, falling back to the next only if a chunk is still too big: (1) double newline → (2) single newline → (3) sentence boundaries → (4) words → (5) characters.
- **Semantic chunking** — groups content by meaning (embedding similarity), giving the best context-preservation and highest-quality RAG results, at the cost of being slower and producing variable-sized chunks.

**Q11: How do you calculate chunk overlap?**

A common starting point is **10–20% overlap** between consecutive chunks (e.g., if chunks are 500 tokens, overlap the last 50–100 tokens of one chunk into the start of the next). This prevents information sitting right at a chunk boundary from being lost entirely from both chunks' context.

**Q12: How would you chunk structured data (tables, JSON) vs. unstructured data (PDFs, text)?**

See the "chunking strategies by data type" reference later in this section — in short: structured data (tables, JSON) needs chunking that preserves the schema/relationships (e.g., repeating headers per chunk, one JSON object per chunk), while unstructured data (PDFs, free text) relies more on layout/structure-aware or semantic chunking, since there's no explicit schema to lean on.

**Q13: What is sliding window chunking? When and why would you use it?**

Instead of cutting text into isolated, non-overlapping blocks, you create **overlapping segments** — each window shares some content with its neighbors. This is essentially the mechanism behind chunk overlap (Q11): it's used specifically to avoid losing context/information that would otherwise fall right at a hard chunk boundary.

**Q14: Bi-encoder vs. cross-encoder?**

- **Bi-encoder** — encodes the query and document **independently** into separate embeddings, then computes similarity (e.g., cosine similarity) between them. Since document embeddings can be precomputed and indexed ahead of time, retrieval at query time is fast.
- **Cross-encoder** — takes the query and document **together** as a single input to the model, so it can directly capture interactions between them. More accurate, since it isn't limited to comparing two independently-compressed vectors, but much slower — it can't be precomputed, since it needs both the specific query and document at the same time.

**Q15: Given 1 million documents and 1,000 queries, compare the computational cost of bi-encoder vs. cross-encoder retrieval.**

- **Bi-encoder:** embed all 1M documents **once**, offline, at indexing time (O(N) total, amortized). Embed each of the 1,000 queries at query time (O(Q)). Compare each query against the index using approximate nearest-neighbor search (e.g., HNSW), roughly O(Q log N) rather than a brute-force O(Q × N) — very tractable at this scale.
- **Cross-encoder:** to find the best matches for a query using a cross-encoder *directly against the full corpus*, you'd need to score **every query-document pair jointly** — 1,000 × 1,000,000 = **1 billion** full transformer forward passes. This is computationally infeasible at this scale.
- **This is exactly why cross-encoders are used only for reranking a small candidate set** (e.g., the top 50–100 results a bi-encoder/hybrid search already narrowed down), not as the primary retrieval method over a full corpus — reranking 1,000 queries × 100 candidates each = 100,000 passes, a very different (and tractable) order of magnitude.

**Q16: Why is reranking with cross-encoders necessary, and what accuracy improvement can you expect?**

Necessary because the initial retrieval pass (bi-encoder or BM25) is optimized for *speed* across a huge corpus, which means it trades away some precision — it's comparing independently-computed representations rather than directly modeling query-document interaction. A cross-encoder reranking pass recovers that precision on a small candidate set. The actual accuracy improvement is dataset- and task-dependent, but relevance-ranking metrics (NDCG, MRR) commonly show a **meaningful double-digit percentage relative improvement** from adding a reranking step — treat any specific number as illustrative rather than universal, and validate empirically on your own data.

**Q17: What is ColBERT, and how does it balance bi-encoder speed with cross-encoder accuracy?**

Covered in detail in Part 2 (Reranking Approaches) — in short: it encodes query and document into **per-token** vectors (rather than one vector each), matches them via "late interaction" (MaxSim per query token, summed), giving meaningfully better accuracy than a plain bi-encoder while remaining faster/cheaper than a full cross-encoder, since document token embeddings can still be precomputed.

**Q18: How do you evaluate the retrieval component separately from the generation component?**

Use **retrieval-only metrics** against a labeled set of (query → known-relevant document) pairs, independent of what the LLM eventually generates: **precision@k, recall@k, MRR (Mean Reciprocal Rank), NDCG**, and the context precision/recall metrics covered earlier in this doc. This isolates whether retrieval is doing its job — if these metrics are strong but end-to-end answer quality is still poor, the problem is in generation/prompting, not retrieval (and vice versa).

**Q19: Explain the two-stage retrieval architecture, and why reranking improves results despite adding latency.**

- **Stage 1 (fast, coarse):** retrieve a broad candidate set (e.g., top 50–100) using a cheap method — BM25, bi-encoder/vector search, or hybrid — across the *entire* corpus.
- **Stage 2 (slow, precise):** rerank *only* that small candidate set using a more expensive, more accurate method (cross-encoder, ColBERT, or LLM-based reranking).

The latency added is bounded, because stage 2 only ever processes the small candidate set, never the full corpus — so the precision gain (stage 2 catches relevant documents stage 1 ranked too low, or filters out false positives stage 1 let through) comes at a small, fixed additional cost rather than scaling with corpus size.

**Q20: Compare different reranking strategies: cross-encoder, LLM-based, and ColBERT.**

Covered in detail in Part 2 (Reranking Approaches — Comparison table) — in short: cross-encoder is the most accurate "classic" option but can't be precomputed; ColBERT is a middle ground (more accurate than plain dense retrieval, faster than cross-encoder, at the cost of storage overhead); LLM-based reranking has the strongest reasoning ability but the highest latency/cost.

**Q21: How do you decide the number of candidates to rerank (the "k" value)?**

A tradeoff: a larger k means more reranker calls (higher latency/cost), but too small a k risks the true relevant document not even being in the candidate set to begin with (since it depends on stage-1 recall). A common practical range is reranking the **top 50–100** candidates down to a final **top 5–10** passed to the LLM. Tune empirically — measure, on a labeled dataset, how often the actually-relevant document appears within the top-k of your stage-1 retrieval, and set k comfortably above that recall point.

**Q22: How do you handle images in a RAG system?**

Two main approaches (also covered in the Azure AI Search section for a platform-specific example):
- **Image verbalization** — use a vision-capable model to generate a text description/caption of the image at ingestion time, then embed and index that description like any other text.
- **Multimodal embeddings** — use a model (e.g., CLIP-style) that embeds both images and text into the **same vector space**, so a text query can be compared directly against image embeddings via vector similarity, without a separate captioning step.

**Q23: What is cosine similarity, and how does it relate to embeddings?**

Measures the **angle** between two vectors in high-dimensional space, ignoring their magnitude:

`cosine similarity = (A · B) / (|A| × |B|)`

where `A · B` is the dot product and `|A|`, `|B|` are the vector magnitudes (norms).

- **1** → vectors point in exactly the same direction (maximally similar).
- **0** → vectors are orthogonal (no similarity).
- **-1** → vectors point in exactly opposite directions.

**Q24: What is a vector database?**

A database purpose-built to store and search **embeddings** — numerical representations of text, images, or other data — enabling similarity search based on semantic meaning rather than exact keyword matches.

**Q25: Why cosine similarity over Euclidean distance?**

- Cosine similarity measures **direction**, ignoring magnitude — so it's not thrown off by embedding vectors having different lengths/scales (e.g., due to document length or other factors influencing magnitude), focusing purely on semantic orientation.
- Euclidean distance **is** sensitive to magnitude, which can distort similarity judgments in high-dimensional embedding spaces.
- Many embedding models are explicitly trained using a cosine-similarity-based objective (contrastive learning), making cosine the "native" metric they were optimized for.
- Caveat worth knowing: if embeddings are **normalized to unit length** first, cosine similarity and Euclidean distance become mathematically monotonically related (they rank results identically) — so the choice can matter less than expected once normalization is applied.

**Q26: What is a Recurrent Neural Network (RNN)?**

A neural network architecture where each time step's output is carried forward and used as input for the next time step — a sequential chain of operations. This contrasts with **Transformers**, which process all input tokens **simultaneously** via self-attention, rather than one at a time. Because of their sequential structure, RNNs struggle with long-range dependencies (information from early in a long sequence tends to degrade by the time it reaches later steps) — one of the key motivations behind the Transformer architecture.

**Q27: What is text tokenization?**

Splitting text into smaller units, called tokens — the fundamental preprocessing step before any of that text can be fed into a neural network.

**Q28: What are the different types of tokenization?**

- **Character-level** — breaks text into individual characters. Simple, but hard for the model to learn meaningful representations from, since a single character carries very little semantic information (weaker performance).
- **Word-level** — splits into whole words. Easier for the model to learn meaningfully, but results in a very large vocabulary (and struggles with words never seen during training).
- **Subword-level** — the common middle ground (see Q9): rare words get split into smaller meaningful pieces, common words stay whole.

**Q29: What is token indexing?**

Converting each token into an integer ID — the numeric representation the model's embedding layer actually operates on.

**Q30: Explain the Transformer architecture.**

Introduced in the paper *"Attention Is All You Need,"* designed to process sequences via **self-attention**, making it well suited to tasks requiring understanding of text and the relationships between its words.

Three primary architectural variants:

*(Note: the encoder-decoder description here has been corrected — translation is actually the classic encoder-decoder use case, not a decoder-only one.)*

- **Encoder-only** (e.g., BERT) — processes the input sequence as a whole to build a rich understanding of it, then makes a prediction *about* that input. Used for tasks needing overall meaning of a text, like sentiment analysis or classification — not for generating new text token-by-token.
- **Decoder-only** (e.g., GPT-style models) — takes the input sequence and **autoregressively generates new tokens**, one at a time, each new token attending only to the tokens that came before it (causal/masked self-attention). This is the architecture behind most modern chat-style LLMs.
- **Encoder-decoder** (e.g., the original Transformer, T5) — the **encoder** processes the full input sequence into a rich internal representation; the **decoder** then generates the output sequence, attending both to previously generated output tokens *and* to the encoder's representation of the input. **Machine translation is the classic example of this architecture** — the encoder reads the full source sentence, and the decoder generates the translated sentence using that encoded representation.

**Core building blocks, common across variants:**
- **Text embedding** — text is tokenized and converted to token IDs; token IDs alone don't capture any relationship between words, so embeddings convert them into dense vectors that do.
- **Positional encoding** — since attention has no inherent sense of token order, a position vector (generated via sine/cosine functions) is added to each token's embedding before it enters the transformer, giving the model information about each token's position in the sequence.
- **Multi-head attention** — runs several attention mechanisms in parallel, each able to specialize in a different kind of relationship between tokens, then combines them for a richer, multi-perspective understanding of each token.
- **Feed-forward network** — after attention gathers context, a feed-forward network processes each token individually for deeper, per-token representation.
- **Prediction head** — translates the transformer's final output into a probability distribution over every token in the vocabulary, used to select the next generated token.

**Q31: What is fine-tuning?**

The process of making a model proficient at a particular task by further training it on task-specific data. **Downsides:** computationally expensive (requires significant compute resources for the model's parameter updates), needs to be re-run periodically to incorporate new/updated data, and requires meaningful technical expertise to do well.

**Q32: Deterministic vs. stochastic sampling?**

- **Deterministic (greedy) sampling** — generates text without randomness, always selecting the single highest-probability token at each step.
- **Stochastic sampling** — introduces randomness into token selection (e.g., via temperature, top-k, or top-p sampling — covered in Q34), producing more varied output across runs.

**Q33: What are offline vs. online evaluation metrics for LLMs?**

**Offline evaluation** (before deployment, against a fixed dataset):
- **Perplexity** — measures how well the model predicts the next word in a text; lower perplexity means the model assigns higher probability to the actual next token.
- **Exact match @ n** — measures how often an n-word generated phrase exactly matches the corresponding n-word phrase in ground truth.

**Online evaluation** (in production, on real traffic):
- User engagement metrics — usage rate, average task completion time, system response time, feedback rate.
- Human evaluation.
- Safety evaluation — identifying the risk of the model generating harmful content.

**Q34: Top-k vs. top-p sampling?**

- **Top-k sampling** — every candidate next token has a probability; tokens are sorted in descending order of probability, and only the top **k** tokens are considered for sampling.
- **Top-p (nucleus) sampling** — tokens are sorted the same way, but instead of a fixed count, the smallest possible set of tokens whose **cumulative probability exceeds a threshold p** is considered.

**Q35: What is prompt engineering?**

Guiding a general-purpose LLM to produce a specific desired output purely through how the prompt is designed, without modifying the model itself. **Pros:** ease of use, cost-effective (no training required). **Cons:** response consistency depends heavily on prompt design — small prompt changes can meaningfully shift output.

Key techniques:
- **Chain-of-thought prompting** — asking the model to reason through a problem step by step before giving a final answer, rather than jumping straight to a conclusion.
- **Few-shot prompting** — giving the model a few example input-output pairs before the actual query, to demonstrate the desired pattern.
- **Role-specific prompting** — having the model adopt a specific role/persona to generate more appropriate, context-fitting responses.
- **User context** — including specific user information in the prompt so the model's output is tailored accordingly.

**Q36: What is RAG (Retrieval-Augmented Generation)?**

Instead of relying solely on an LLM's general training knowledge, RAG retrieves data from relevant external sources and feeds it to the LLM at inference time.
- **Retrieval** — takes the user's original prompt, finds the most relevant information from an external source, and returns it as context.
- **Generation** — the LLM uses the user's prompt *and* the retrieved information together to generate its response.

**Q37: Rule-based vs. AI-based document parsing?**

- **Rule-based parsing** — relies on predefined rules based on an expected document layout/structure. Struggles badly if a document doesn't match the expected format.
- **AI-based parsing** — uses techniques like object detection and OCR (optical character recognition) to identify document elements more flexibly (tools like `dedoc`, Unstructured, or Azure Document Intelligence are examples), handling varied/unexpected layouts far better than a fixed rule set.

**Q38: When should you stop/skip reranking?**

Covered earlier (Part 2, "When should you skip reranking") — in short: latency-sensitive applications where speed matters more than precision, exact-term/keyword-only queries where semantic reranking adds no value, or resource-constrained situations where "good enough" retrieval is acceptable.

**Q39: What is an embedding?**

A dense vector representation of a discrete object (a word, sentence, image, etc.) that captures its semantic meaning as a point in a high-dimensional space — objects with similar meaning end up with vectors that are close together in that space.

**Q40: What are good chunking strategies for different data types?**

- **Tables** — repeat headers in every chunk so context isn't lost; consider converting rows to natural-language sentences, or treating each row as its own document, depending on the use case.
- **PDFs** — use object detection and OCR to identify document elements (text blocks, tables, images) before chunking, since raw PDF text extraction often loses structure.
- **Code** — chunk by logical unit (function, class), not by fixed character count, to keep code semantically coherent.
- **Markdown/HTML** — keep the relevant heading as a prefix in every chunk, so context about "where" a chunk sits in the document isn't lost. For HTML specifically, strip tags first, then chunk by semantic section.
- **Images** — generate text descriptions using vision models; for charts/diagrams specifically, dedicated extraction tools (that pull out data/labels, not just a general caption) tend to work better.
- **JSON** — flatten by key-value pairs, or treat each JSON object as one chunk, depending on how self-contained each object is.

**Q41: What metrics are used for generation evaluation?**

- **ROUGE** — measures n-gram overlap between generated text and a reference (originally for summarization).
- **BLEU** — measures n-gram precision against a reference (originally for translation).
- **BERTScore** — uses contextual embeddings to measure semantic similarity between generated and reference text, rather than exact n-gram overlap — generally a better fit for judging meaning-level correctness than ROUGE/BLEU alone.
- **Faithfulness** — is the generated content actually grounded in/supported by the source material (covered extensively earlier in this doc for RAG specifically).
- **Groundedness** — closely related to faithfulness; whether claims in the output can be traced back to real supporting evidence.

**Q42: What is embedding caching?**

Storing computed vector representations (embeddings) so they don't need to be recomputed every time the same content is encountered again — saves both cost (embedding API calls aren't free) and latency, especially valuable for content that rarely changes (e.g., a static document corpus) or for repeated/similar queries (semantic caching, covered earlier in this doc).

---

# Part 10: LLM Evaluation — LangSmith / OpenEvals Deep Dive

**Q1: What is LLM evaluation?**

The process of measuring the quality of responses generated by an LLM — determining whether the model produces correct, relevant, faithful, and useful answers. There are two broad types:
- **Offline evaluation** — uses a predefined dataset with expected outputs.
- **Online evaluation** — measures production metrics such as user feedback, latency, and engagement.

**Q2: Why do we need evaluation?**

- Measure model quality.
- Compare prompt or model versions.
- Detect hallucination.
- Validate retrieval quality in RAG.
- Prevent regressions after deployment.

**Q3: Difference between offline and online evaluation?**

Same distinction as Q1: offline runs against a curated dataset with known-correct expected outputs (used before a release); online measures real production signals (user feedback, latency, engagement) on live traffic, generally without a ground-truth reference available for every interaction.

**Q4: What is an evaluation dataset?**

A collection of representative test cases, each typically containing: a user input, an expected output, and optional metadata. For RAG applications specifically, it may also include the retrieved context.

**Q5: What is a reference output?**

The expected/ground-truth answer used to compare against the model's generated answer — written or verified by a human.

**Q6: What is a code evaluator (in LangSmith)?**

A Python function that computes a metric deterministically — it can use plain Python logic, regular expressions, API calls, or even call another LLM internally.

```python
def exact_match(outputs, reference_outputs):
    return outputs["answer"] == reference_outputs["answer"]
```

**Q7: What is an LLM-as-a-Judge evaluator?**

Uses another LLM to score the quality of generated responses, rather than comparing outputs with fixed deterministic logic. The judge LLM is prompted to evaluate dimensions like correctness, faithfulness, relevance, and completeness.

**Q8: When would you use an LLM judge instead of Python logic?**

- **Python logic** works well for deterministic checks: exact match, regex validation, numeric comparison.
- **LLM judges** are better for subjective/qualitative dimensions that don't reduce to a simple rule: summary quality, answer completeness, faithfulness, writing quality.

**Q9: What metrics are commonly used in RAG evaluation?**

Correctness, faithfulness, context relevance, context recall, context precision, answer relevance.

**Q10: What is correctness?**

*(Fixed a typo — the source question read "what is correct?")* Measures whether the generated answer matches the reference answer. Usually requires a reference output to compare against.

**Q11: What is faithfulness?**

Measures whether the answer is fully supported by the retrieved context — this is the metric that detects hallucination specifically.

Example:
```
Context: Kerala has 14 districts.
Answer:  Kerala has 15 districts.
```
Correctness is low here (assuming the reference also says 14), *and* faithfulness is also low — the answer directly contradicts the context it was supposed to be grounded in.

**Q12: What is context relevance?**

Measures whether the retrieved documents are relevant to the user's question — this evaluates **retrieval quality**, not answer quality.

**Q13: What is context recall?**

Measures whether the retriever fetched **all** the information actually required to answer the question (as opposed to context precision, which measures whether what *was* retrieved is relevant).

**Q14: Which metrics require reference answers?**

- **Require a reference:** correctness, context recall.
- **Usually don't require a reference:** faithfulness (compares the answer against the retrieved context, not a ground truth), context relevance (compares retrieved context against the question, not a ground truth).

**Q15: What is `create_llm_as_judge()`?**

A helper function (provided by OpenEvals or LangSmith) that automatically constructs an LLM-based evaluator. Internally it: builds a prompt, calls the judge LLM, parses its output, and returns a structured evaluation result.

**Q16: What does `@traceable` do?**

*(Correction: this is a **LangSmith** SDK decorator, not something specific to LangGraph — though it's commonly used inside LangChain/LangGraph applications for exactly this purpose.)* Wraps a function so that its inputs, outputs, latency, errors, and metadata are automatically recorded/traced — the core mechanism behind LangSmith's tracing and debugging capability.

**Q17: What is a composite evaluator?**

Combines multiple evaluation metrics into a single overall score, typically via a weighted sum:

```
Correctness      = 0.95
Faithfulness      = 0.90
Context Relevance = 0.85

Overall = 0.4×0.95 + 0.3×0.90 + 0.3×0.85
```

**Q18: Why use a composite evaluator?**

Instead of examining multiple metrics individually, a composite evaluator produces a single score representing overall system quality — making it much easier to compare experiments/versions at a glance.

**Q19: What is pairwise evaluation?**

Compares two experiments/outputs **directly against each other**, rather than scoring each independently. The judge receives Answer A and Answer B, and decides: A is better, B is better, or it's a tie.

**Q20: When should pairwise evaluation be used?**

Useful when comparing prompt versions, model versions, chunking strategies, embedding models, or retrieval algorithms — anywhere you want to know "which is better," not just "how good is each in isolation."

**Q21: Why not rely only on correctness scores?**

Two responses can both be independently "correct" while differing meaningfully in completeness, clarity, fluency, or helpfulness. Pairwise evaluation captures these qualitative differences that a single correctness score would miss entirely.

**Q22: How would you evaluate a RAG system in production?**

A common workflow:

```
Production logs
      ↓
Find representative / failure cases
      ↓
Human review
      ↓
Add good cases to the evaluation dataset
      ↓
Run offline evaluation before the next release
      ↓
Deploy
      ↓
Monitor production
      ↓
New failure cases surface
      ↓
Improve the evaluation dataset
```

In practice, this splits into two layers:
- **Offline evaluation dataset** — curated tester questions plus representative real-user questions sampled from production logs, with trusted reference answers. Used before releases to compare model, prompt, chunking, retrieval, and embedding changes.
- **Online production monitoring** — real user inputs, generated outputs, retrieved contexts, latency, tokens, errors, and user feedback. Evaluators like faithfulness can still be run on sampled production traces even without a reference answer, since faithfulness compares against retrieved context rather than ground truth.

You generally maintain a curated evaluation dataset rather than treating every production interaction as evaluation data — the dataset is deliberately built and reviewed, not just a raw log dump.

**Q23: What should an evaluation dataset contain?**

User question, reference answer, generated answer, retrieved context, and optional metadata.

**Q24: Difference between Code Evaluator, LLM-as-a-Judge, Composite Evaluator, and Pairwise Evaluator?**

| Evaluator | Purpose |
|---|---|
| **Code Evaluator** | A Python function that computes a metric deterministically |
| **LLM-as-a-Judge** | Uses an LLM to score a response on subjective/qualitative dimensions |
| **Composite Evaluator** | Combines multiple metrics into one overall score |
| **Pairwise Evaluator** | Compares outputs from two experiments and selects the preferred one |

**Q25: What should the target function return in OpenEvals/LangSmith evaluation?**

The target function represents the application being evaluated — it receives the input from the evaluation dataset, actually runs the application, and returns whatever outputs the evaluators need.

For a RAG application, it typically returns:
- **Answer** — the LLM-generated response.
- **Context** — the documents/chunks the RAG system retrieved.

```python
def target(inputs: dict) -> dict:
    result = query(inputs["question"])

    return {
        "answer": result["answer"],
        "context": result["context"]
    }
```

These outputs then feed different metrics: correctness (generated answer vs. reference answer), faithfulness (generated answer vs. retrieved context), context relevance (question vs. retrieved context), context recall (retrieved context vs. reference answer). The target function isn't limited to `answer`/`context` — it can return additional fields (sources, document IDs, etc.) if specific evaluators need them.