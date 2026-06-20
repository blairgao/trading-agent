# Investment Agents

> ⚠️ **Status: Planning / Pre-implementation.** Architecture and scope below are a working draft, not yet built.

## Overview

This project builds a system of AI agents that automate the full investment research-to-execution loop:

1. **Research agents** — gather and synthesize information (news, filings, earnings calls, on-chain/market data, macro indicators) into investment theses.
2. **Market data agents** — track real-time prices, volume, volatility, and other signals across target markets.
3. **Execution agents** — place and manage trades based on approved strategies, within defined risk limits.

The goal is for these agents to propose — and, where authorized, execute — investment strategies on our behalf, with a harness for running them reliably and an eval system for measuring whether they're actually any good.

## Goals

- Automate repetitive investment research so a human can review theses instead of compiling them from scratch.
- Maintain a live picture of relevant markets without manual polling.
- Let agents propose concrete strategies (entry/exit conditions, position sizing, rationale) for human approval.
- Optionally allow approved strategies to be executed automatically within strict guardrails.
- Continuously evaluate agent output against real outcomes so we know when to trust (or stop trusting) them.

## Non-goals (for now)

- Fully autonomous trading with no human in the loop.
- Guaranteed profitability — this is a research and automation tool, not a trading signal vendor.
- Coverage of every asset class on day one; start narrow, expand deliberately.

## High-Level Architecture

```
┌────────────────────┐     ┌────────────────────┐     ┌────────────────────┐
│  Research Agents   │     │ Market Data Agents │     │  Execution Agents  │
│ - news/filings     │────▶│ - real-time feeds  │────▶│ - order placement  │
│ - sentiment        │     │ - indicators       │     │ - position mgmt    │
│ - thesis drafting  │     │ - alerts           │     │ - risk limits      │
└──────────┬─────────┘     └──────────┬─────────┘     └──────────┬─────────┘
           │                          │                          │
           └──────────────────────────┴──────────────────────────┘
                        ▼                          ▼
             ┌────────────────────┐     ┌────────────────────┐
             │   Agent Harness    │     │    Eval System     │
             │ (orchestration,    │     │ (backtests,        │
             │  memory, tools)    │     │  live scoring)     │
             └────────────────────┘     └────────────────────┘
```

### Agent Harness

The harness is the runtime that all agents share:

- **Orchestration** — scheduling, agent-to-agent handoffs, retries.
- **Tool/data access** — standardized interfaces to market data APIs, broker/exchange APIs, news/search, and internal storage.
- **Memory** — short-term (per-session) and long-term (positions, past theses, outcomes) state.
- **Guardrails** — hard limits on trade size, leverage, asset universe, and an explicit human-approval step before any live execution (at least initially).

### Eval System

Before agents are trusted with real capital, every proposed strategy and trade is scored:

- **Backtesting** — replay historical data to check whether a strategy/agent would have been profitable and well-calibrated.
- **Live shadow mode** — agents run and propose trades without executing, scored against what actually happened.
- **Calibration tracking** — compare agent-stated confidence to realized outcomes over time.
- **Regression tests** — fixed scenarios/prompts to catch when a model or prompt change degrades agent quality.

## Planned Stack

- **Language**: Python
- **Agent framework**: LangChain / LangGraph (tentative — open to alternatives as the design firms up)
- **Data sources**: TBD (market data API, news/filings API, broker/exchange API)
- **Storage**: TBD (likely Postgres for trade/thesis history, vector store for research memory)
- **Eval/backtesting**: TBD

## Repository Structure (proposed)

```
.
├── agents/
│   ├── research/        # research & thesis-generation agents
│   ├── market_data/      # real-time data ingestion & monitoring agents
│   └── execution/         # order placement & position management agents
├── harness/                 # orchestration, memory, tool interfaces, guardrails
├── evals/                    # backtests, scoring, regression tests
├── strategies/             # proposed/approved strategy definitions
└── README.md
```

## Risk & Safety Notes

- No agent executes a real trade without passing through the eval system and an explicit approval gate, at least until the system has a proven track record.
- All agents operate within hard-coded position size, asset, and risk limits — not just prompted limits.
- This system handles real money; treat credentials, API keys, and execution paths with the same care as production financial infrastructure.

## Status / Next Steps

- [ ] Define target markets/assets for v1 (e.g. equities, crypto, etc.)
- [ ] Pick data providers and broker/exchange API
- [ ] Build harness skeleton (orchestration + tool interfaces)
- [ ] Build one research agent end-to-end as a proof of concept
- [ ] Stand up backtesting eval before any live or shadow trading
- [ ] Define approval workflow for human-in-the-loop execution

---
*This README is a living document — update it as architecture decisions are made.*