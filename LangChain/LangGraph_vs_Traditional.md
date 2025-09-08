---
# Traditional Software vs AI/Multi-Agent Workflows

## 1. Traditional Software Methods/Functions (Deterministic)

- You write a function → it executes the same way every time.
- Good for:
	- API calls
	- Data processing
	- Workflows with strict rules

## 2. AI/Multi-Agent Workflows (Probabilistic)

- An LLM response can vary run to run.
- State needs to be tracked:
	- Memory of conversation
	- Task progress
	- Errors
	- Retries
- Agents may hand off control to other agents dynamically (not pre-coded).

> You need something more flexible than just “call function A then function B.”

---

## 3. Why LangGraph’s State Graph Matters

LangGraph’s state graph lets you:

- **Track context over time:** An LLM may call multiple tools, and you need a structured state store to remember results.
- **Branch dynamically:** Depending on what the LLM outputs, the graph can route execution down different paths.
- **Handle retries/failures gracefully:** Built-in to graph execution (whereas in plain code, you’d have to write extra orchestration).
- **Coordinate multiple agents:** Each node in the graph can be an “agent” with its own role, tools, and responsibilities.
- **Human-in-the-loop:** Graph state can pause and wait for human validation before resuming.

> That’s hard to do cleanly with just methods and functions.

---

## 4. Feeding Tasks into an LLM vs LangGraph

dynamic routing,
clean agent orchestration,
resumability and observability.
### If you just feed a list of tasks into an LLM:

- You rely on the LLM to keep track of everything in its hidden context.
- There’s no explicit state you can inspect or debug.
- If something fails halfway, you’d likely restart from scratch.

### With LangGraph:

- You can see and inspect the workflow state at each step.
- You can resume, reroute, or replay tasks.

---

## 5. Concrete Example: AI Travel Booking Agent

**Traditional approach:**

- Write functions to call Expedia API, weather API, etc.
- Write if/else logic for flight vs hotel vs car rental.
- You must anticipate every possible flow yourself.

**LangGraph approach:**

1. User asks “Plan me a trip.”
2. State graph routes to “Planner agent” node.
3. Planner decides: needs flights → hands off to Flight agent node.
4. Flight agent calls APIs, updates state.
5. Graph routes back to Planner, then to Hotel agent.
6. If Flight agent fails, graph retries or routes to fallback agent.

Here, the graph orchestrates both deterministic code and LLM decisions, with explicit state you can inspect.

> We could write it all as traditional software with methods, but we’d lose:
> - Structured state management
> - Dynamic routing
> - Clean agent orchestration
> - Resumability and observability

LangGraph gives you those AI-native workflow guarantees, while still letting you drop in normal functions and APIs.