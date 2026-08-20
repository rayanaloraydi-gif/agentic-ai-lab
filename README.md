# Stadium Booking Agent (Capstone Project)

**Author:** ريان العريدي
**Programme:** Agentic AI Systems (SDAIA Academy)
**Cohort Dates:** August 2026
**Track:** Track A (Multi-Agent Routing)

This repository contains the final Capstone project for the [SDAIA Academy](https://github.com/SDAIAAcademy). 
## How to Run
1. Open the `.ipynb` notebook in Google Colab.
2. Run the first cell to install all required dependencies (`langchain`, `langgraph`, etc.).
3. Run the second cell. A secure prompt will appear asking for your `LANGCHAIN_API_KEY` and `GROQ_API_KEY`. Paste your real keys there (they will not be saved in the code).
4. Run the remaining cells sequentially to initialize the RAG pipeline, the Multi-Agent Router, and execute the tests.
## 1. Workflow Pattern & Routing
This system implements the **Routing Pattern**. A supervisor LLM uses `with_structured_output` to classify user queries and route them to one of three tasks: RAG search, Booking tool, or Escalation.

## 2. RAG Pipeline
We chose a **Hybrid RAG approach** to handle dynamic stadium policies. Documents are chunked using `RecursiveCharacterTextSplitter`, embedded, and stored in a `FAISS` vector store.

## 3. LangGraph & Error Handling
Built entirely on the Functional API (`@task` and `@entrypoint`). We implemented two error strategies:
1. `RetryPolicy` on the routing task.
2. A Fallback task (try/except block) to catch unhandled errors.

## 4. LangSmith Observability
The trace showed that the `rag_search` step dominated the latency, taking around 1.2s. The total run cost was low. A possible improvement is caching frequent policy questions to reduce latency.
