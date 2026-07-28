# 🎯 AI & Backend Interview Cheatsheet

**Last-Minute Review Guide | One-Line Definitions | 30-Minute Study Session**

---

## 🤖 AI & LLM Core Concepts

### Fundamental Terms
| Term | Definition |
|------|------------|
| **LLM** | Large Language Model; neural network trained on vast text data to predict and generate text sequences. |
| **Token** | Smallest unit of text processed by LLMs (word, subword, or character); input/output measured in tokens. |
| **Transformer** | Neural architecture using self-attention to process sequences in parallel; foundation for modern LLMs. |
| **Attention Mechanism** | Allows model to focus on relevant parts of input; computes weighted importance of each token. |
| **Embedding** | Numerical vector representation of text; similar meanings have similar vectors (semantic proximity). |

### Model Training & Tuning
| Term | Definition |
|------|------------|
| **Pre-training** | Initial training on massive unlabeled data to learn general language patterns; expensive, done once. |
| **Fine-tuning** | Training pre-trained model on task-specific data; adapts model for specific domain/task efficiently. |
| **Prompt Engineering** | Crafting input text to guide model behavior without retraining; includes prompts, examples, instructions. |
| **Few-shot Learning** | Providing few examples in prompt to teach model new task without fine-tuning. |
| **Zero-shot Learning** | Model performs task without examples; relies on pre-training knowledge and task description. |
| **In-context Learning** | Model learns from examples within the prompt to perform new tasks during inference. |

### Retrieval & Generation
| Term | Definition |
|------|------------|
| **RAG** | Retrieval-Augmented Generation; combines retrieval from external documents with LLM generation for accuracy. |
| **Vector Database** | Database storing embeddings; enables semantic search and similarity matching (e.g., Pinecone, Weaviate). |
| **Semantic Search** | Finding documents based on meaning, not keywords; uses embeddings and similarity metrics. |
| **Chunking** | Splitting documents into smaller pieces for efficient retrieval and indexing. |
| **Retrieval Ranking** | Scoring and ordering retrieved documents by relevance to query (BM25, semantic similarity). |
| **Context Window** | Maximum number of tokens model can process; limits input size and retrieved context. |

### Advanced Techniques
| Term | Definition |
|------|------------|
| **HyDE** | Hypothetical Document Embeddings; generates hypothetical answer before retrieval for better queries. |
| **RAG-Fusion** | Converts single query into multiple search queries for diverse results and better coverage. |
| **Step-Back Prompting** | Ask abstract question first to understand higher-level concept, then answer specific question. |
| **CRAG** | Corrective RAG; verifies retrieved documents and refines retrieval if confidence is low. |
| **Chain-of-Thought** | Prompting model to show step-by-step reasoning instead of jumping to answer. |
| **Self-Consistency** | Generate multiple reasoning paths and use majority vote for more reliable answers. |

---

## 🤖 Agentic AI & Multi-Agent Systems

| Term | Definition |
|------|------------|
| **AI Agent** | Autonomous system that perceives environment, makes decisions, and takes actions to achieve goals. |
| **Agentic Workflow** | Loop of perception → reasoning → action → observation; agent interacts with environment iteratively. |
| **Tool/Function Calling** | LLM decides which tool/function to call; enables agents to interact with external systems. |
| **ReAct** | Reasoning + Acting; pattern where agents alternate between reasoning and tool use. |
| **Plan-Execute Pattern** | Agent creates plan first, then executes steps; better for complex multi-step tasks. |
| **Multi-Agent System** | Multiple agents cooperating/competing; can specialize, handle complex problems, improve robustness. |
| **Agent Communication** | Agents exchange information via message passing, shared memory, or hierarchical protocols. |
| **State Management** | Tracking agent memory, beliefs, goals across interactions; persistent context for learning. |

---

## 📊 Evaluation & Monitoring

| Term | Definition |
|------|------------|
| **BLEU Score** | Metric comparing generated text to reference; higher = better, commonly used for machine translation. |
| **ROUGE Score** | Recall-Oriented Understudy for GIST Evaluation; measures overlap between generated and reference text. |
| **Perplexity** | How "surprised" model is by test data; lower = better, inverse probability of held-out data. |
| **Hallucination** | LLM generates plausible-sounding but false information; major challenge for factual accuracy. |
| **Factuality Check** | Verification that generated text matches ground truth; essential for RAG and fact-based systems. |
| **Human Evaluation** | Assessing quality via humans; expensive but crucial for nuanced qualities like coherence and tone. |
| **A/B Testing** | Comparing two versions with real users; gold standard for measuring actual user preference. |
| **Latency** | Time to generate response; critical for real-time applications and user experience. |
| **Throughput** | Number of requests processed per unit time; important for scalability. |
| **Cost per Token** | Price of input/output tokens; impacts profitability and system sustainability. |

---

## 🔒 Safety, Security & Compliance

| Term | Definition |
|------|------------|
| **Prompt Injection** | Malicious input tries to override model instructions or leak hidden information. |
| **Guardrails** | Rules/filters preventing model from harmful outputs (profanity, illegal content, misinformation). |
| **Content Filtering** | Detecting and blocking inappropriate content in input/output. |
| **PII (Personally Identifiable Information)** | Data identifying individuals (names, emails, SSN); must be protected/anonymized. |
| **Data Privacy** | Safeguarding user data; compliance with GDPR, CCPA, etc. |
| **Model Bias** | Systematic errors favoring certain groups; arises from training data or design. |
| **Fairness** | Ensuring model treats all groups equitably; auditing and mitigating bias. |
| **Explainability** | Understanding why model makes certain decisions; important for trust and compliance. |
| **Model Cards** | Documentation of model capabilities, limitations, training data, and recommended uses. |

---

## 🏗️ System Design & Architecture

| Term | Definition |
|------|------------|
| **Scalability** | System's ability to handle increased load; horizontal (more machines) vs. vertical (stronger machine). |
| **Availability** | Percentage of time system is operational; measured as uptime (e.g., 99.9% = 3-nines). |
| **Latency** | Time delay between request and response; lower is better for user experience. |
| **Throughput** | Number of requests/tasks processed per unit time; measures system capacity. |
| **Load Balancing** | Distributing requests across servers to optimize resource use and prevent bottlenecks. |
| **Caching** | Storing frequently used data in fast storage; reduces computation and improves response time. |
| **Rate Limiting** | Restricting number of requests per user/time; protects against abuse and overload. |
| **Database Indexing** | Creating index structures for fast data retrieval; critical for query performance. |
| **Partitioning/Sharding** | Splitting data across multiple databases; enables horizontal scaling. |
| **Replication** | Copying data across servers; improves availability and fault tolerance. |

---

## 💾 Backend & .NET Concepts

### C# Fundamentals
| Term | Definition |
|------|------------|
| **Async/Await** | Non-blocking I/O pattern; allows multiple operations without blocking threads. |
| **LINQ** | Language-Integrated Query; provides SQL-like syntax for querying collections and data. |
| **Delegates** | Type-safe function pointers; allows passing methods as parameters. |
| **Events** | Publish-subscribe pattern; enables loose coupling between components. |
| **Generics** | Type-parameterized classes/methods; provides type safety and code reuse. |
| **Reflection** | Inspecting and manipulating types/members at runtime; powerful but slow. |
| **Dependency Injection** | Providing dependencies externally; improves testability and loose coupling. |

### ASP.NET Core & Architecture
| Term | Definition |
|------|------------|
| **Middleware** | Software component in request pipeline; examples: logging, authentication, error handling. |
| **Middleware Pipeline** | Ordered sequence of middleware; request flows through each in order, response returns in reverse. |
| **MVC** | Model-View-Controller; separation of data (model), presentation (view), and logic (controller). |
| **Dependency Injection Container** | Manages object creation and dependency resolution; built into ASP.NET Core. |
| **Routing** | Mapping HTTP requests to controllers/handlers; matches URL patterns to actions. |
| **Model Binding** | Converting HTTP request data to C# objects; handles deserialization automatically. |
| **Validation** | Checking data correctness; includes attribute-based and custom validators. |
| **Authorization** | Determining if authenticated user can access resource; role-based, policy-based, or claims-based. |

### Authentication & Security
| Term | Definition |
|------|------------|
| **JWT (JSON Web Token)** | Stateless token containing claims; includes header, payload, signature for secure transmission. |
| **OAuth 2.0** | Authorization protocol allowing users to grant app access without sharing passwords. |
| **OpenID Connect** | Layer on OAuth 2.0 for authentication; adds identity verification and user info endpoint. |
| **Claims** | Statements about user (identity, roles, permissions); carried in tokens. |
| **Scope** | Permissions requested from user in OAuth flow; limits app access to specific resources. |
| **Refresh Token** | Long-lived token used to obtain new short-lived access token without user re-authentication. |

### Data Access & Performance
| Term | Definition |
|------|------------|
| **ORM (Object-Relational Mapping)** | Framework mapping objects to database tables; examples: EF Core, Dapper. |
| **Entity Framework Core** | Microsoft's ORM; enables LINQ queries against databases; supports migrations. |
| **N+1 Query Problem** | Fetching parent then individual queries for each child; very inefficient, solved by eager loading. |
| **Eager Loading** | Loading related entities with main entity; prevents N+1 problem (e.g., .Include()). |
| **Lazy Loading** | Loading related entities only when accessed; can cause N+1 if not careful. |
| **Connection Pooling** | Reusing database connections; reduces overhead of connection creation. |
| **Caching Strategy** | In-memory (fast, single-machine) vs. distributed (multiple machines, persistent). |
| **Distributed Caching** | Caching across multiple servers; enables cache sharing and persistence (e.g., Redis). |

### Messaging & Async Patterns
| Term | Definition |
|------|------------|
| **Message Queue** | Asynchronous communication between services; decouples sender and receiver. |
| **Pub/Sub** | Publisher sends message once; multiple subscribers receive independently. |
| **Service Bus** | Managed message broker; provides reliable delivery, routing, and scaling (e.g., Azure Service Bus). |
| **Dead-Letter Queue** | Queue for messages that fail processing; enables debugging and recovery. |
| **Message Delivery Guarantee** | At-least-once (duplicates possible), at-most-once (loss possible), exactly-once (hardest). |
| **Backpressure** | Mechanism to slow down producer if consumer can't keep up; prevents queue overload. |

---

## ☁️ Azure & Cloud Services

| Term | Definition |
|------|------------|
| **Azure AI Foundry** | Platform for building/deploying AI models; integrates LLMs, tools, and infrastructure. |
| **Copilot Studio** | Low-code platform for building AI agents; visual designer for workflows and integrations. |
| **Model Context Protocol (MCP)** | Standard for AI agents to interact with external tools/data sources; standardizes tool integration. |
| **Azure Cognitive Search** | Managed search service; combines keyword search with semantic search via embeddings. |
| **Azure OpenAI Service** | Microsoft's managed access to OpenAI models; provides API with compliance and security. |
| **Managed Identity** | Azure feature for apps to authenticate without storing credentials; improves security. |
| **Key Vault** | Secure storage for secrets, keys, certificates; prevents hardcoding sensitive data. |
| **App Configuration** | Centralized config management; enables dynamic updates without redeployment. |
| **Application Insights** | Monitoring and diagnostics for apps; tracks performance, errors, and usage. |

---

## 📐 Database & Data Structures

| Term | Definition |
|------|------------|
| **ACID** | Atomicity (all-or-nothing), Consistency (valid state), Isolation (no interference), Durability (persistent). |
| **NoSQL** | Non-relational databases (document, key-value, graph); schema-flexible, horizontally scalable. |
| **Document Database** | Stores data as documents (JSON/BSON); flexible schema (e.g., MongoDB). |
| **Graph Database** | Optimized for relationships; excels at complex queries involving connections (e.g., Neo4j). |
| **Time-Series Database** | Optimized for time-stamped data; efficient for metrics, logs, monitoring. |
| **Index** | Data structure speeding up lookups; trades space for query speed. |
| **B-Tree** | Self-balancing tree; commonly used for database indexes. |
| **Hash Table** | Map keys to values; O(1) average lookup; used for caching and quick access. |

---

## 🔄 Design Patterns

| Term | Definition |
|------|------------|
| **Singleton** | Ensures single instance of class; used for shared resources (logging, config). |
| **Factory** | Creates objects without specifying exact classes; enables flexible instantiation. |
| **Observer** | Notifies multiple objects of state change; loose coupling between publisher/subscribers. |
| **Strategy** | Defines family of algorithms; allows switching at runtime without modifying client. |
| **Decorator** | Adds behavior to object without modifying original; wrapper pattern for extensibility. |
| **Adapter** | Makes incompatible interfaces compatible; wrapper for legacy/third-party code. |
| **Facade** | Provides simplified interface to complex subsystem; hides complexity. |
| **Repository** | Abstraction for data access; centralizes query logic and enables easy testing. |
| **Unit of Work** | Manages transaction across multiple repositories; ensures data consistency. |

---

## 🎓 Soft Skills & Interview Tips

| Term | Definition |
|------|------------|
| **STAR Method** | Structure for behavioral answers: Situation, Task, Action, Result. |
| **Trade-offs** | Discuss pros/cons of design choices; show balanced thinking. |
| **Scalability Questions** | Ask clarifying questions about load, growth, constraints before designing. |
| **Latency vs. Throughput** | Latency = individual response time; throughput = requests per second. |
| **Consistency vs. Availability** | CAP theorem: can't have all three simultaneously (Consistency, Availability, Partition-tolerance). |
| **Time Complexity** | Big O notation; O(1), O(n), O(log n), O(n²), etc. |
| **Space Complexity** | Memory usage; measured in Big O similarly. |

---

## 🚀 Quick Reference: Interview Patterns

### When Asked "Design a System":
1. **Clarify requirements** (functional, non-functional, constraints)
2. **High-level architecture** (components and interactions)
3. **Detailed deep-dive** (database choice, caching, messaging)
4. **Scale analysis** (bottlenecks, optimization)
5. **Trade-offs** (consistency vs. availability, latency vs. throughput)

### When Asked "Explain a Concept":
1. **Definition** (what is it?)
2. **Use case** (when/why use it?)
3. **How it works** (mechanism or example)
4. **Alternatives** (other approaches?)
5. **Pros/cons** (trade-offs)

### When Asked "Debug This":
1. **Reproduce** the issue
2. **Isolate** the component
3. **Root cause** analysis
4. **Solution** design
5. **Prevention** for future

---

## 📚 Study Tips

✅ **Do's:**
- Review this sheet daily (5-15 minutes)
- Create flashcards for weak areas
- Explain terms to others
- Connect terms to concepts
- Practice with real scenarios

❌ **Don'ts:**
- Memorize definitions verbatim
- Skip understanding how terms relate
- Ignore hands-on practice
- Review only day before interview
- Use cheatsheet as only study resource

---

## 🔗 Cross-References

**For detailed explanations, see:**
- **LLM Deep Dive:** `LLM Interview Questions.pdf`
- **Gen AI Breadth:** `Gen AI Questions.md`
- **System Design:** `generative-ai-system-design-interview-1_compress.pdf`
- **Agent Architectures:** `Deep-Understanding-of-AI-Agents-Li-Bojie-v1.1.pdf`
- **.NET Backend:** `Azure & .NET C# Interview Prep Notes.md`

---

## 💡 Pro Tips

1. **30-minute routine:** Review 1 category daily (10 terms × 3 minutes each)
2. **Teach someone:** Explain terms to friend without looking; best way to learn
3. **Real-world examples:** Connect each term to actual projects/companies
4. **Mock interview:** Use this sheet to brainstorm answers, not to copy
5. **Update as needed:** Add personal notes and links to resources you found helpful

---

**Last Updated:** July 2026  
**Best Used:** 7 days before interview + daily during preparation  
**Estimated Review Time:** 30-60 minutes

---

**Good luck with your interview! 🚀**
