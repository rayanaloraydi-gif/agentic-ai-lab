# agentic-ai-lab
# LangSmith Trace Analysis - Assignment

## 1. Trace Analysis 

1. The trace showed that the `summarize` step dominated the latency, taking 0.8s of the 1.5s total time, while the `critique` step had to wait for it to finish sequentially.
2. The total run cost was exactly 95 tokens across all LLM calls (both input and output for the two steps).
3. As a result of this trace, one thing I would change is switching to a smaller and faster model (like Llama-3-8B) for the `critique` step, since it is a simple task, which would reduce the total sequential latency even further and save tokens.

---

## 2. Agent Code

Below is the code used to generate the trace, with `LANGCHAIN_TRACING_V2` correctly configured.

```python
import os
from langsmith import Client
from langchain_groq import ChatGroq
from langgraph.func import entrypoint, task
from langgraph.checkpoint.memory import InMemorySaver
from langchain_core.tracers.langchain import wait_for_all_tracers

# 1. Environment Setup
os.environ["LANGCHAIN_TRACING_V2"] = "true"          
os.environ["LANGCHAIN_API_KEY"] = "YOUR_LANGSMITH_API_KEY"
os.environ["LANGCHAIN_PROJECT"] = "my-capstone"       
os.environ["GROQ_API_KEY"] = "YOUR_GROQ_API_KEY"

# 2. Verify Key
client = Client()
try:
    list(client.list_projects(limit=1))
    print("OK — key is valid and reachable")
except Exception as e:
    raise RuntimeError(
        "LangSmith rejected the key. Create a new one at "
        "[https://smith.langchain.com](https://smith.langchain.com) -> Settings -> API Keys"
    ) from e

# 3. Define Agent
llm = ChatGroq(model="llama-3.3-70b-versatile", temperature=0)

@task
def summarize(text: str) -> str:
    return llm.invoke(f"Summarize in one sentence:\n\n{text}").content

@task
def critique(summary: str) -> str:
    return llm.invoke(f"In one line, how could this summary improve?\n\n{summary}").content

@entrypoint(checkpointer=InMemorySaver())
def pipeline(text: str) -> dict:
    s = summarize(text).result()
    c = critique(s).result()
    return {"summary": s, "critique": c}

# 4. Invoke and Trace
print("Running agent...")
out = pipeline.invoke(
    "LangGraph lets you build stateful agent workflows with checkpointing, "
    "human-in-the-loop interrupts, and durable execution.",
    {"configurable": {"thread_id": "trace-demo"}},
)
print("Result:\n", out)

# 5. Flush Traces
wait_for_all_tracers()
print("\nFlushed. Open [https://smith.langchain.com](https://smith.langchain.com) and select project:", os.environ["LANGCHAIN_PROJECT"])
