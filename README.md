# Superior Skills

OpenClaw-compatible trading skill and execution layer for agents.

Turn natural-language trading intent into structured strategies, backtests, and live deployment on [Superior Trade](https://www.superior.trade/).

## Why this repo exists

Most AI agents can talk about trading. Very few can reliably turn a trading idea into validated, executable code.

This repository is the public home of the Superior Trade skill file and supporting documentation for how agents use Superior Trade for strategy creation, backtesting, and deployment.

It is designed for:

- **OpenClaw users** looking to add trading capabilities to their agent
- **Agent builders** who need a trading execution layer
- **Developers** integrating with the Superior Trade API
- **Platform operators** evaluating skill-based trading infrastructure

## What Superior Trade does

Superior Trade helps users and agents:

- Turn trading intent into structured strategy logic
- Define entry, exit, sizing, and safeguards
- Backtest strategies before deployment
- Deploy through managed execution infrastructure

The goal is turning an idea into something clear enough to test, review, and run — not just code generation.

## Why this matters

In live markets, a good idea can still fail because the path from idea to execution is weak. The real challenge is the full pipeline:

**idea → strategy logic → validation → deployment**

Superior Trade is built for that pipeline.

## Quickstart

### Install the skill

**Option 1 — Prompt your agent:**

```
Get strategy trading skills from https://superior.trade/SKILL.md
```

**Option 2 — Install from ClawHub:**

Find the skill on [clawhub/superior-ai/superior-trade](https://clawhub.com/superior-ai/superior-trade) and install it from there.

**Option 3 — Manual:**

Copy [`SKILL.md`](./SKILL.md) from this repo directly into your agent's skill directory.

### First prompts to try

- `Backtest a BTC strategy with RSI`
- `Create a SOL breakout strategy with ATR stop loss`
- `Deploy a short strategy on low volume altcoins`
- `DCA into GOLD when price is low`
- `Create a long/short strategy and show me the backtest before deployment`

## What is in this repo

| File | Description |
|---|---|
| [`SKILL.md`](./SKILL.md) | The published skill file — full API reference, strategy authoring guide, and agent operating rules |
| [`README.md`](./README.md) | This file |

## Core capabilities

### Intent → Strategy
Describe a trading objective in plain language and convert it into explicit strategy logic.

### Backtest before deploy
Strategies are validated before touching live capital.

### Managed live execution
Deploy through Superior Trade's execution layer instead of relying on fragile local tooling.

### Agent-native integration
Built for agent workflows — the skill file is structured so models can read and use it directly.

## Workflow

The default path is:

1. Describe the objective
2. Structure the strategy
3. Define risk and safeguards
4. Backtest the setup
5. Review before promotion
6. Deploy only when ready

## Related repositories

| Repo | Description |
|---|---|
| [superior-examples](https://github.com/Superior-Trade/superior-examples) | Setup prompts, copy-paste workflows, and practical trading use cases |
| [superior-api](https://github.com/Superior-Trade/superior-api) | API documentation, request examples, and test cases |
| [superior-mcp](https://github.com/Superior-Trade/superior-mcp) | MCP integration and open-source connector layer |

## Safety philosophy

The default operating pattern is:

**draft → backtest → review → deploy**

No live deployment starts without explicit user confirmation. See the Safety section in [`SKILL.md`](./SKILL.md) for full details.

## Links

- Website: https://www.superior.trade/
- OpenClaw page: https://www.superior.trade/openclaw
- Skill file: https://superior.trade/SKILL.md
- GitHub: https://github.com/Superior-Trade

## Disclaimer

Trading involves risk. Backtests do not guarantee future performance. Users remain responsible for strategy choice, deployment decisions, exchange connectivity, and capital risk.
