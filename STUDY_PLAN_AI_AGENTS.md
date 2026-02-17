# Generative AI Agent System Design — 60-Day Study Plan

> From zero to designing and building production AI agent systems. **1 hour/day.**  
> Each day has: what to learn, the best free resource, and a hands-on task.

---

## Phase 1: AI & LLM Foundations (Days 1–10)

Understand how language models work before building on top of them.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 1 | What are neural networks — intuition, not math | [3Blue1Brown: Neural Networks (video series)](https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi) | Run a simple neural net in PyTorch — classify MNIST digits |
| 2 | Word embeddings — how machines understand text | [Jay Alammar: The Illustrated Word2Vec](https://jalammar.github.io/illustrated-word2vec/) | Use `gensim` to load Word2Vec embeddings — find similar words, do `king - man + woman` |
| 3 | Attention mechanism — the core of transformers | [Jay Alammar: The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) | Visualize attention weights using [BertViz](https://github.com/jessevig/bertviz) on a sample sentence |
| 4 | How GPT works — decoder-only transformers | [Jay Alammar: How GPT3 Works](https://jalammar.github.io/how-gpt3-works-visualizations-animations/) + [Andrej Karpathy: Let's build GPT (video)](https://www.youtube.com/watch?v=kCc8FmEb1nY) | Read through [nanoGPT](https://github.com/karpathy/nanoGPT) — understand the training loop |
| 5 | Tokenization — BPE, SentencePiece, tiktoken | [Andrej Karpathy: Let's build the GPT Tokenizer (video)](https://www.youtube.com/watch?v=zduSFxRajkE) + [HuggingFace: Tokenizers](https://huggingface.co/docs/tokenizers/en/index) | Compare tokenization of the same paragraph across GPT-4 (`tiktoken`), Llama (`sentencepiece`), and BERT |
| 6 | Prompt engineering — zero-shot, few-shot, chain-of-thought | [Prompt Engineering Guide](https://github.com/dair-ai/Prompt-Engineering-Guide) | Write 10 prompts for the same task — compare: zero-shot vs few-shot vs CoT on GPT-4/Claude |
| 7 | LLM APIs — OpenAI, Anthropic, open-source | [OpenAI API Docs](https://platform.openai.com/docs/api-reference) + [Anthropic API Docs](https://docs.anthropic.com/en/docs/initial-setup) | Call OpenAI & Anthropic APIs from Python — stream responses, handle errors, count tokens |
| 8 | Open-source LLMs — Llama, Mistral, Phi, Qwen | [Ollama](https://github.com/ollama/ollama) + [HuggingFace Open LLM Leaderboard](https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard) | Install Ollama, run Llama 3.1 8B locally, compare output quality with GPT-4 on 5 prompts |
| 9 | Fine-tuning basics — LoRA, QLoRA | [HuggingFace PEFT Docs](https://huggingface.co/docs/peft/en/index) + [Sebastian Raschka: Practical Tips for Finetuning LLMs](https://magazine.sebastianraschka.com/p/practical-tips-for-finetuning-llms) | Fine-tune a small model (Phi-3-mini) on a custom Q&A dataset using QLoRA + HuggingFace |
| 10 | Quantization & inference optimization | [TheBloke's Quantization Guide](https://huggingface.co/TheBloke) + [llama.cpp](https://github.com/ggerganov/llama.cpp) | Quantize a model to GGUF Q4 with `llama.cpp` — benchmark speed vs full precision |

---

## Phase 2: Retrieval-Augmented Generation — RAG (Days 11–22)

The most practical pattern for production AI: grounding LLMs in your data.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 11 | RAG architecture — why, when, how | [RAG Paper (Lewis et al.)](https://arxiv.org/abs/2005.11401) + [LangChain: RAG Quickstart](https://python.langchain.com/docs/tutorials/rag/) | Draw the full RAG pipeline on paper: `Query → Embed → Retrieve → Augment → Generate` |
| 12 | Text embeddings — sentence-transformers, OpenAI embeddings | [SBERT.net](https://www.sbert.net/) + [MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard) | Generate embeddings for 100 documents using `sentence-transformers` — compute cosine similarity |
| 13 | Vector databases — concepts (HNSW, IVF, PQ) | [Pinecone: What is a Vector Database?](https://www.pinecone.io/learn/vector-database/) + [FAISS Wiki](https://github.com/facebookresearch/faiss/wiki) | Install FAISS, index 10K embeddings, run ANN search — compare `Flat` vs `IVF` vs `HNSW` |
| 14 | Vector DB tools — Chroma, Weaviate, Qdrant, Pgvector | [Chroma](https://github.com/chroma-core/chroma) + [Qdrant](https://github.com/qdrant/qdrant) + [pgvector](https://github.com/pgvector/pgvector) | Spin up Chroma + Qdrant in Docker, index the same docs in both — compare query latency |
| 15 | Document chunking strategies | [Unstructured: Chunking Best Practices](https://unstructured.io/blog/chunking-for-rag-best-practices) + [Greg Kamradt: 5 Levels of Text Splitting](https://www.youtube.com/watch?v=8OJC21T2SL4) | Implement 4 splitting strategies (fixed, recursive, semantic, document-aware) — compare retrieval quality |
| 16 | **Build Day**: Basic RAG pipeline | — | Build an end-to-end RAG app: load PDFs → chunk → embed → store in Chroma → query with GPT-4 |
| 17 | Advanced retrieval — hybrid search (dense + sparse) | [Pinecone: Hybrid Search](https://www.pinecone.io/learn/hybrid-search-intro/) | Implement BM25 (sparse) + dense embedding search — combine scores with Reciprocal Rank Fusion |
| 18 | Query transformation — HyDE, multi-query, step-back | [LangChain: Query Transformations](https://python.langchain.com/docs/how_to/query_transformations/) | Implement HyDE (Hypothetical Document Embeddings): generate a fake answer, embed that, retrieve |
| 19 | Re-ranking — cross-encoders, Cohere Rerank | [Cross-Encoders (SBERT)](https://www.sbert.net/examples/applications/cross-encoder/README.html) + [Cohere Rerank](https://docs.cohere.com/docs/reranking) | Add a cross-encoder re-ranker to your day-16 pipeline — measure retrieval precision improvement |
| 20 | Structured & multi-modal RAG | [LlamaIndex: Multi-Modal RAG](https://docs.llamaindex.ai/en/stable/examples/multi_modal/multi_modal_pdf_tables/) | Build RAG over a PDF with tables and images — use `unstructured` for parsing, GPT-4V for images |
| 21 | RAG evaluation — RAGAS, faithfulness, relevance | [RAGAS](https://github.com/explodinggradients/ragas) + [RAG Triad of Metrics](https://www.trulens.org/trulens/getting_started/core_concepts/rag_triad/) | Evaluate your day-16 pipeline with RAGAS: context precision, faithfulness, answer relevancy |
| 22 | **Build Day**: Production RAG system | — | Rebuild your RAG with: hybrid search + re-ranking + query transformation + RAGAS eval — compare metrics with day-16 |

---

## Phase 3: AI Agents — Tool Use & Reasoning (Days 23–38)

From passive Q&A to autonomous agents that take actions.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 23 | What are AI agents — ReAct, reasoning + acting | [ReAct Paper (Yao et al.)](https://arxiv.org/abs/2210.03629) + [Lilian Weng: LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) | Implement a bare ReAct loop in Python (~50 lines): `Think → Act → Observe → repeat` |
| 24 | Function/tool calling — OpenAI, Anthropic tool use | [OpenAI: Function Calling](https://platform.openai.com/docs/guides/function-calling) + [Anthropic: Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | Give GPT-4 tools for weather, calculator, and web search — watch it choose the right tool |
| 25 | LangChain agents — architecture & components | [LangChain: Agents](https://python.langchain.com/docs/concepts/agents/) + [LangChain GitHub](https://github.com/langchain-ai/langchain) | Build a LangChain agent with: Wikipedia search + Python REPL + calculator tools |
| 26 | LangGraph — stateful multi-step agent workflows | [LangGraph Docs](https://langchain-ai.github.io/langgraph/) + [LangGraph GitHub](https://github.com/langchain-ai/langgraph) | Build a LangGraph agent with branching: `research → analyze → write report` with human-in-the-loop |
| 27 | CrewAI — multi-agent collaboration | [CrewAI](https://github.com/crewAIInc/crewAI) | Create a 3-agent crew: Researcher + Writer + Editor — generate a blog post collaboratively |
| 28 | AutoGen — conversational multi-agent | [AutoGen](https://github.com/microsoft/autogen) | Build a coding assistant: User ↔ Planner Agent ↔ Coder Agent ↔ Reviewer Agent |
| 29 | Planning & decomposition — task breakdown | [HuggingGPT Paper](https://arxiv.org/abs/2303.17580) + [Plan-and-Solve Paper](https://arxiv.org/abs/2305.04091) | Implement a planning agent that breaks "Plan a trip to Japan" into subtasks and executes each |
| 30 | Memory systems — short-term, long-term, episodic | [Lilian Weng: Agents (Memory section)](https://lilianweng.github.io/posts/2023-06-23-agent/#memory) + [Mem0](https://github.com/mem0ai/mem0) | Add persistent memory to your agents using Mem0 — make the agent remember past conversations |
| 31 | Code generation agents — writing & executing code | [Open Interpreter](https://github.com/OpenInterpreter/open-interpreter) + [E2B Code Interpreter](https://github.com/e2b-dev/code-interpreter) | Build an agent that writes Python code, runs it in a sandbox (E2B), and iterates on errors |
| 32 | Web browsing agents | [Browser Use](https://github.com/browser-use/browser-use) + [Playwright](https://github.com/microsoft/playwright-python) | Build an agent that can search Google, navigate to pages, extract data — using Playwright |
| 33 | **Build Day**: Personal research agent | — | Build: `User query → Web search → Scrape top 5 pages → Summarize → Generate report with citations` |
| 34 | Agent evaluation & benchmarks | [AgentBench](https://github.com/THUDM/AgentBench) + [SWE-bench](https://github.com/princeton-nlp/SWE-bench) | Evaluate your day-33 agent on 10 research questions — measure accuracy, hallucination rate, latency |
| 35 | Guardrails & safety — preventing harmful outputs | [Guardrails AI](https://github.com/guardrails-ai/guardrails) + [NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) | Add NeMo Guardrails to your agent: block jailbreaks, enforce topic boundaries, validate output format |
| 36 | Human-in-the-loop patterns | [LangGraph: Human-in-the-Loop](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/) | Add approval checkpoints to your agent: pause before executing actions, let user confirm |
| 37 | Structured output — JSON mode, Pydantic, Instructor | [Instructor](https://github.com/jxnl/instructor) + [Outlines](https://github.com/dottxt-ai/outlines) | Use Instructor to extract structured data (Pydantic models) from unstructured text — 100% type-safe |
| 38 | **Build Day**: Full-featured AI agent | — | Build a customer support agent: RAG over docs + ticket creation tool + escalation logic + guardrails + memory |

---

## Phase 4: Production AI System Design (Days 39–50)

Scaling, observing, and hardening AI systems for production.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 39 | LLM system architecture patterns | [a]( https://github.com/Shubhamsaboo/awesome-llm-apps) + [Chip Huyen: Building LLM Applications for Production](https://huyenchip.com/2023/04/11/llm-engineering.html) | Draw architecture diagrams for 3 patterns: simple RAG, agentic RAG, multi-agent system |
| 40 | LLM gateway & routing — managing multiple models | [LiteLLM](https://github.com/BerriAI/litellm) + [Portkey](https://github.com/Portkey-AI/gateway) | Set up LiteLLM as a proxy — route requests to OpenAI/Anthropic/Ollama with fallback |
| 41 | Prompt management & versioning | [PromptLayer](https://github.com/MagnivOrg/prompt-layer-library) + [LangSmith](https://docs.smith.langchain.com/) | Version 5 prompt variants in LangSmith, A/B test them, track which performs best |
| 42 | Caching LLM responses — semantic cache | [GPTCache](https://github.com/zilliztech/GPTCache) | Add GPTCache to your RAG pipeline — cache similar queries, measure cost/latency reduction |
| 43 | Observability for LLMs — tracing, cost tracking | [LangFuse](https://github.com/langfuse/langfuse) + [Phoenix (Arize)](https://github.com/Arize-ai/phoenix) | Integrate LangFuse into your agent — trace every LLM call, tool use, retrieval, with costs |
| 44 | LLM evaluation at scale — LLM-as-judge | [OpenAI Evals](https://github.com/openai/evals) + [DeepEval](https://github.com/confident-ai/deepeval) | Build an eval suite with DeepEval: test correctness, hallucination, bias on 50 test cases |
| 45 | Serving LLMs — vLLM, TGI, Ollama | [vLLM](https://github.com/vllm-project/vllm) + [TGI](https://github.com/huggingface/text-generation-inference) | Deploy Llama 3.1 8B with vLLM — benchmark throughput (tokens/sec) with concurrent requests |
| 46 | Streaming & async — SSE, response streaming | [OpenAI: Streaming](https://platform.openai.com/docs/api-reference/streaming) | Build a FastAPI endpoint that streams LLM responses via SSE — add a React frontend that renders token-by-token |
| 47 | Cost optimization — token budgeting, model cascading | [Not Diamond](https://github.com/Not-Diamond/notdiamond-python) + [Martian Router](https://docs.withmartian.com/) | Implement model cascading: try small model first → escalate to GPT-4 only if confidence is low |
| 48 | Security — prompt injection, data leakage | [OWASP Top 10 for LLMs](https://owasp.org/www-project-top-10-for-large-language-model-applications/) + [Rebuff](https://github.com/protectai/rebuff) | Test your agent against 10 prompt injection attacks — add input/output sanitization |
| 49 | Deployment — Docker, K8s, serverless | [Modal](https://github.com/modal-labs/modal-client) + [BentoML](https://github.com/bentoml/BentoML) | Package your RAG pipeline as a BentoML service — deploy with Docker |
| 50 | **Build Day**: Production-grade AI system | — | Deploy: `FastAPI + LiteLLM + RAG (Qdrant) + LangFuse tracing + Redis semantic cache + Guardrails` |

---

## Phase 5: Advanced Agent Architectures (Days 51–55)

Cutting-edge patterns.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 51 | Agentic RAG — retrieval as a tool, self-correcting | [LlamaIndex: Agentic RAG](https://docs.llamaindex.ai/en/stable/examples/agent/agentic_rag/) | Build an agent that decides when to retrieve, evaluates relevance, and re-queries if needed |
| 52 | Graph RAG — knowledge graphs + LLMs | [Microsoft GraphRAG](https://github.com/microsoft/graphrag) | Run GraphRAG on a set of documents — compare answers with standard vector RAG |
| 53 | Multi-modal agents — vision, audio, video | [GPT-4 Vision](https://platform.openai.com/docs/guides/vision) + [Whisper](https://github.com/openai/whisper) | Build an agent that takes a screenshot, describes it, and takes actions based on what it sees |
| 54 | Autonomous coding agents — Devin-style | [SWE-agent](https://github.com/princeton-nlp/SWE-agent) + [Aider](https://github.com/Aider-AI/aider) | Run SWE-agent or Aider on a real GitHub issue — watch how it navigates a codebase |
| 55 | Agent-to-agent protocols — MCP, A2A | [Model Context Protocol (MCP)](https://github.com/modelcontextprotocol) + [Google A2A](https://github.com/google/A2A) | Build an MCP server that exposes your database as a tool — connect it to Claude Desktop |

---

## Phase 6: Design & Build Real Systems (Days 56–60)

Put it all together.

| Day | System to Design | Reference | Activity |
|---|---|---|---|
| 56 | Design a customer support AI (Intercom-like) | [Intercom's Resolution Bot](https://www.intercom.com/ai-bot) | Design: RAG over help docs + ticket creation + escalation + sentiment detection + analytics |
| 57 | Design a code generation platform (Cursor-like) | [Cursor Blog](https://cursor.sh/blog) + [Continue.dev](https://github.com/continuedev/continue) | Design: code context retrieval + multi-file edit + codebase indexing + streaming + caching |
| 58 | Design an enterprise search assistant (Glean-like) | [Danswer](https://github.com/danswer-ai/danswer) | Design: multi-source connectors + unified embedding + permissions + query routing + re-ranking |
| 59 | Design a document processing pipeline (legal/finance) | [Docugami](https://www.docugami.com/platform) | Design: OCR + table extraction + entity recognition + structured output + audit trail |
| 60 | **Mock Design Interview** | [LLM System Design Template](https://github.com/seanpm2001/AI-2027) | Do a 45-min AI system design mock: state requirements, draw architecture, discuss trade-offs |

---

## Essential Open-Source Projects to Study

| Category | Project | Why Study It |
|---|---|---|
| **LLM Framework** | [LangChain](https://github.com/langchain-ai/langchain) | Chains, agents, tools, memory abstractions |
| **Agent Framework** | [LangGraph](https://github.com/langchain-ai/langgraph) | Stateful, cyclical agent graphs with persistence |
| **Agent Framework** | [CrewAI](https://github.com/crewAIInc/crewAI) | Multi-agent role-based collaboration |
| **RAG Framework** | [LlamaIndex](https://github.com/run-llama/llama_index) | Data ingestion, indexing, retrieval, querying |
| **Vector DB** | [Chroma](https://github.com/chroma-core/chroma) | Simple, developer-friendly embedding database |
| **Vector DB** | [Qdrant](https://github.com/qdrant/qdrant) | Production vector search with filtering |
| **LLM Serving** | [vLLM](https://github.com/vllm-project/vllm) | PagedAttention, continuous batching, high throughput |
| **Local LLM** | [Ollama](https://github.com/ollama/ollama) | Run LLMs locally with a single command |
| **Structured Output** | [Instructor](https://github.com/jxnl/instructor) | Type-safe LLM outputs with Pydantic |
| **Observability** | [LangFuse](https://github.com/langfuse/langfuse) | LLM tracing, evaluation, prompt management |
| **Evaluation** | [RAGAS](https://github.com/explodinggradients/ragas) | RAG evaluation metrics |
| **Guardrails** | [NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) | Safety rails for LLM applications |
| **LLM Gateway** | [LiteLLM](https://github.com/BerriAI/litellm) | Unified API for 100+ LLM providers |
| **Search + RAG** | [Danswer](https://github.com/danswer-ai/danswer) | Full open-source enterprise RAG system |
| **Graph RAG** | [Microsoft GraphRAG](https://github.com/microsoft/graphrag) | Knowledge-graph-augmented RAG |
| **Coding Agent** | [SWE-agent](https://github.com/princeton-nlp/SWE-agent) | Autonomous code generation & bug fixing |
| **MCP** | [Model Context Protocol](https://github.com/modelcontextprotocol) | Standard protocol for tool/context integration |

---

## Key Papers to Read (1 per week)

| Week | Paper | Key Idea |
|---|---|---|
| 1 | [Attention Is All You Need (2017)](https://arxiv.org/abs/1706.03762) | Transformer architecture |
| 2 | [RAG (Lewis et al., 2020)](https://arxiv.org/abs/2005.11401) | Retrieval-augmented generation |
| 3 | [ReAct (Yao et al., 2022)](https://arxiv.org/abs/2210.03629) | Reasoning + acting in agents |
| 4 | [LoRA (Hu et al., 2021)](https://arxiv.org/abs/2106.09685) | Efficient fine-tuning |
| 5 | [Chain-of-Thought (Wei et al., 2022)](https://arxiv.org/abs/2201.11903) | Step-by-step reasoning |
| 6 | [Toolformer (Schick et al., 2023)](https://arxiv.org/abs/2302.04761) | LLMs learning to use tools |
| 7 | [Constitutional AI (Bai et al., 2022)](https://arxiv.org/abs/2212.08073) | Self-improvement & safety |
| 8 | [GraphRAG (Microsoft, 2024)](https://arxiv.org/abs/2404.16130) | Knowledge graphs for RAG |

---

## Daily Routine (1 Hour)

```
[00:00 - 00:05]  Review yesterday's notes (5 min)
[00:05 - 00:30]  Read/watch the day's resource (25 min)
[00:30 - 01:00]  Hands-on build task (30 min)
```

> **Rule**: No day is reading-only. You must build, run code, or experiment every single day.  
> **Tip**: Keep a `learning-journal.md` — write 3 bullet points of what you learned each day.
