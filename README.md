# Conversational Credit Negotiation — Agent Flow

Reference implementation of a **multi-agent conversational flow for debt negotiation**, built on [CrewAI Flow](https://docs.crewai.com/concepts/flows).

This repository is the reasoning core of a three-part reference stack: this flow, a [FastAPI middleware](https://github.com/gsolmyrg/conversational_credit_negotiation_middleware) that exposes it over HTTP, and a [Streamlit app](https://github.com/gsolmyrg/conversational_credit_negotiation_app) for simulating sessions.

> All personas and financial data in this repository are synthetic. No client data, corpora, or production configuration is included.

## What it demonstrates

**Router-based state machine instead of a prompt chain.** The flow uses `@start`, `@router`, `@listen` and `and_` to branch on real state rather than hoping the model follows instructions. Persona missing? It routes to a dedicated branch instead of hallucinating a customer.

**Two specialized agents rather than one general prompt.** A classification agent labels every incoming message (`initial_engagement`, `in_scope`, `out_of_scope`, and a deliberate off-topic category), while a negotiation agent builds a plan from the customer persona. Classification and planning run in parallel and converge with `and_` before the response is generated.

**Structured outputs, not string parsing.** Every agent call returns a validated Pydantic model (`ConversationClassification`, `Message`), so downstream code operates on typed data instead of regex over prose.

**Persisted conversation state.** `@persist()` keeps flow state across turns, with explicit history management and a controlled `clear_history` switch — conversation reset is a deliberate operation, not a side effect.

**Guarded tool use.** The agent may query external search for a narrow, well-defined case, then steer the conversation back on topic. The pattern shown is scope control: the tool exists, but the agent is instructed when *not* to keep using it.

**Tone policy as an explicit instruction.** The negotiation prompt states that the agent must not be pushy when the customer gives a compelling reason not to pay. Collections is a regulated, reputationally sensitive domain, and that constraint belongs in the design rather than in a post-hoc filter.

## Architecture

```
ConversationalFlow (persisted state)
  |
  +-- clear_state_if_needed        @start
  +-- check_if_there_is_a_persona  @router
  |     +-- no_persona_found  -> persona_not_found
  |     +-- persona_found     -> generate_plan          (negotiation agent)
  |                          -> classify_user_message  (classification agent)
  +-- respond_to_user              @listen(and_(plan, classification))
  +-- increment_history
  +-- return_response
```

## Layout

```
src/conversational_credit_negotiation_flow/
  agents/     classification_agent.py, debt_negotiation_agent.py
  models/     flow_state.py, persona.py, message.py,
              conversation_classification.py, debt_negotiation.py, option_config.py
  tools/      custom_tool.py
  main.py     flow definition, kickoff and plot entry points
```

## Stack

Python 3.10+ · CrewAI Flow · Pydantic · [uv](https://docs.astral.sh/uv/)

## Running it

```bash
pip install uv
uv sync
cp .env.example .env   # add your model API key
crewai run
```

To render the flow graph:

```bash
uv run plot
```

## Author

**Guilherme Candeloro Padilha** — AI Solutions Architect
[LinkedIn](https://www.linkedin.com/in/guilherme-candeloro-72113426b) · guilherme@aiveon.com
