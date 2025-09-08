---
# Semantic Kernel: Multi-Agent Systems

> Semantic Kernel can be used to develop multi-agent systems, but the way it does it is different from LangGraph or frameworks built with “agents” as a first-class concept.

---

## How Semantic Kernel Handles Multi-Agents

### Plugins/Skills as Agents

In Semantic Kernel, you don’t typically define “agents” directly. Instead, you build plugins (skills)—these can be semantic (prompt-based) or native code. Each plugin can encapsulate an agent-like capability (e.g., **Research Analyst**, **Summarizer**, **Translator**).

- Multiple skills can run side by side, like independent agents.
- They can call each other through the Kernel.

### Planner as Coordinator

The Planner can decompose a user request into a sequence of steps. Those steps may call different skills/agents. So the planner acts like a multi-agent orchestrator.

**Example:**
- Agent A (skill): Fetch market data
- Agent B (skill): Run Monte Carlo simulation
- Agent C (skill): Summarize results in plain English

The Planner wires these together automatically.

### Memory + Context

Semantic Kernel includes long-term memory (embedding-based) and context objects that can be shared across skills. This lets multiple agents/skills “collaborate” with shared state.

### Human-in-the-Loop

Since Semantic Kernel is built for enterprises, you can design flows where one agent hands off to a human or requires approval before another agent continues.

---

## What’s Missing (Compared to LangGraph)

- **No graph-based state machine:** LangGraph lets you explicitly define cycles, conditionals, checkpoints, and multi-agent flows as a graph.
- **Less natural for open-ended agent collaboration:** Semantic Kernel is more about plugins + planner orchestration, not emergent agent dynamics.
- **Multi-agent support is implicit** (through multiple skills/plugins), not first-class (no explicit “agent” object).

---

Each skill in SK behaves like an agent, and the Planner coordinates them.

✅ Bottom line:
Semantic Kernel can definitely support multi-agent patterns, but it’s plugin/skill + planner driven, whereas frameworks like LangGraph provides a more explicit, graph-based agent orchestration model with native support for cycles, streaming, and multi-agent state.

🔹 Architecture Comparison: Semantic Kernel vs. LangGraph (Multi-Agent)
Semantic Kernel (plugin + planner model)        LangGraph (graph + state model)
-----------------------------------------------------------------------------------
[User Request]                                 [User Request]
     |                                               |
     v                                               v
  [Planner] -----------------------------+     [StateGraph Controller]
     |                                    |           |
     v                                    |           v
[MarketDataSkill] (Agent 1)               |    ┌─────────────────────────┐
     |                                    |    |  MarketData Node (Agent1)|
     v                                    |    └─────────────┬───────────┘
[RiskAnalysisSkill] (Agent 2)             |                  |
     |                                    |                  v
     v                                    |    ┌─────────────────────────┐
[ReportingSkill] (Agent 3)                +--->| RiskAnalysis Node (Agent2)|
     |                                         └─────────────┬───────────┘
     v                                                       |
  [Final Report Output]                                       v
                                                        ┌─────────────────────────┐
                                                        | Reporting Node (Agent3) |
                                                        └─────────────┬───────────┘
                                                                      |
                                                                      v
                                                               [Final Report Output]


Semantic Kernel: Planner auto-determines sequence of skills (agents).

LangGraph: Developer explicitly defines nodes (agents) and edges (flow).