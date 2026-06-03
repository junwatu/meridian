# 06. Repo Map

Bab ini adalah peta file utama.

## Runtime Core

| File | Fungsi |
|---|---|
| `index.js` | App bootstrap, cron, REPL, Telegram handlers |
| `agent.js` | LLM ReAct loop dan role-based tool filtering |
| `prompt.js` | System prompt builder |
| `config.js` | Config loader dan deploy amount logic |
| `logger.js` | Daily logs dan action audit |

## Tool System

| File | Fungsi |
|---|---|
| `tools/definitions.js` | Tool schema yang dilihat LLM |
| `tools/executor.js` | Tool dispatcher dan safety layer |
| `tools/screening.js` | Pool discovery, filtering, scoring |
| `tools/dlmm.js` | Meteora DLMM operations |
| `tools/wallet.js` | Wallet balance dan Jupiter swap |
| `tools/token.js` | Token info, holders, audit, narrative |
| `tools/study.js` | Study top LPers |
| `tools/okx.js` | OKX signal/risk integration |
| `tools/chart-indicators.js` | Optional chart indicator checks |
| `tools/agent-meridian.js` | Agent Meridian API helpers |

## State and Memory

| File | Fungsi |
|---|---|
| `state.js` | Local position registry |
| `lessons.js` | Learning/performance system |
| `pool-memory.js` | Per-pool history and cooldowns |
| `decision-log.js` | Structured decision audit |
| `smart-wallets.js` | Smart wallet tracking |
| `token-blacklist.js` | Token blacklist |
| `dev-blocklist.js` | Deployer/dev blacklist |
| `strategy-library.js` | Saved LP strategies |
| `signal-tracker.js` | Discord signal staging |
| `signal-weights.js` | Darwinian signal weights |

## Interfaces

| File | Fungsi |
|---|---|
| `cli.js` | CLI command `meridian` |
| `telegram.js` | Telegram bot send/poll/control |
| `briefing.js` | Daily Telegram briefing |
| `setup.js` | Setup wizard |
| `ecosystem.config.cjs` | PM2 process config |

## Tests

| File | Fungsi |
|---|---|
| `test/test-agent.js` | Agent test |
| `test/test-screening.js` | Screening test |
| `npm run test:syntax` | JS syntax check across repo |

## Runtime Files

Gitignored runtime files:

- `.env`
- `user-config.json`
- `state.json`
- `lessons.json`
- `pool-memory.json`
- `decision-log.json`
- `smart-wallets.json`
- `token-blacklist.json`
- `dev-blocklist.json`
- `signal-weights.json`
- `logs/`

## Where To Start Reading

For architecture:

1. `index.js`
2. `agent.js`
3. `tools/executor.js`
4. `tools/definitions.js`
5. `config.js`

For trading flow:

1. `tools/screening.js`
2. `tools/dlmm.js`
3. `tools/wallet.js`
4. `state.js`
5. `lessons.js`

For operator experience:

1. `telegram.js`
2. `cli.js`
3. `briefing.js`

## Ringkasan

Repo ini cukup modular. Kuncinya adalah memahami bahwa `agent.js` hanya orchestrator; action nyata berada di `tools/`, dan safety utama ada di `tools/executor.js`.

