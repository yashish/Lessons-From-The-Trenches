. Traditional software vs AI/multi-agent workflows

Traditional software methods/functions are deterministic:

You write a function → it executes the same way every time.

Good for things like API calls, data processing, workflows with strict rules.

AI/multi-agent workflows are probabilistic:

An LLM response can vary run to run.

State needs to be tracked (memory of conversation, task progress, errors, retries).

Agents may hand off control to other agents dynamically (not pre-coded).

So you need something more flexible than just “call function A then function B.”
