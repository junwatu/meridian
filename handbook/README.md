# Meridian Handbook

Panduan modular untuk memahami repo Meridian sebagai Node.js full-stack developer yang baru masuk ke dunia Web3.

## Untuk Siapa

Handbook ini ditulis untuk pembaca yang sudah paham konsep umum seperti:

- Node.js runtime
- API client
- environment variables
- background jobs
- CLI
- state management
- logging
- service layer
- safety checks

Tetapi belum terbiasa dengan konsep Web3 seperti wallet, pool, liquidity position, Solana transaction, RPC, dan DeFi risk.

Tujuannya bukan membuat kamu langsung trading. Tujuannya membuat kamu bisa membaca repo ini, memahami arsitekturnya, tahu risiko teknisnya, dan bisa mengembangkan sistem dengan aman.

## Cara Membaca

Kalau kamu baru mulai, baca berurutan:

1. [Orientation](./01-orientation.md)
2. [Web3 Basics for Node Developers](./02-web3-basics-for-node-developers.md)
3. [Problems Solved](./03-problems-solved.md)
4. [System Architecture](./04-system-architecture.md)
5. [Runtime Lifecycle](./05-runtime-lifecycle.md)

Kalau kamu mau langsung kerja di kode, baca:

1. [Repo Map](./06-repo-map.md)
2. [Agent and Tool Design](./07-agent-and-tool-design.md)
3. [Safety and Risk Controls](./08-safety-and-risk-controls.md)
4. [Config and Runtime State](./09-config-and-runtime-state.md)

Kalau kamu mau belajar sistem Web3 agent secara utuh, lanjutkan sampai:

1. [Screening, Deploy, and Management](./10-screening-deploy-management.md)
2. [Outputs, Logs, and Learning](./11-outputs-logs-learning.md)
3. [Limitations and Improvement Ideas](./12-limitations-and-improvements.md)

## Daftar Isi

| Bab | Topik |
|---|---|
| [01](./01-orientation.md) | Apa itu Meridian dan bagaimana memikirkannya sebagai aplikasi Node.js |
| [02](./02-web3-basics-for-node-developers.md) | Dasar Web3 yang perlu dipahami developer backend/full-stack |
| [03](./03-problems-solved.md) | Masalah yang diselesaikan sistem ini |
| [04](./04-system-architecture.md) | Arsitektur besar dan alur data |
| [05](./05-runtime-lifecycle.md) | Startup, cron, screening, management, Telegram, shutdown |
| [06](./06-repo-map.md) | Peta file dan tanggung jawab setiap modul |
| [07](./07-agent-and-tool-design.md) | Desain LLM agent, role, prompt, dan tools |
| [08](./08-safety-and-risk-controls.md) | Safety layer, risk config, dry-run, dan protected actions |
| [09](./09-config-and-runtime-state.md) | Config, `.env`, JSON state, memory, dan lessons |
| [10](./10-screening-deploy-management.md) | Cara pool dicari, deploy dilakukan, dan posisi dikelola |
| [11](./11-outputs-logs-learning.md) | Apa yang dihasilkan sistem: reports, logs, decisions, lessons |
| [12](./12-limitations-and-improvements.md) | Kekurangan, risiko, dan roadmap improvement |

## Mental Model Cepat

Meridian bisa dipahami seperti aplikasi backend event-driven:

```text
Triggers
  -> job orchestration
  -> context builder
  -> LLM decision engine
  -> tool request
  -> safety middleware
  -> external service adapter
  -> persisted local state
```

Dengan padanan yang familiar:

| Meridian | Analogi Node.js/full-stack |
|---|---|
| `index.js` | App bootstrap + scheduler + controller |
| `agent.js` | Orchestrator service |
| `prompt.js` | Policy/context builder |
| `tools/definitions.js` | API contract/schema |
| `tools/executor.js` | Middleware + service dispatcher |
| `tools/*.js` | External service adapters |
| `state.json` | Local app state |
| `lessons.json` | Learning/event history |
| `decision-log.json` | Audit log |
| Telegram | Operator UI |

## Safety Reminder

Repo ini bisa mengirim transaksi nyata jika `DRY_RUN=false`. Perlakukan seperti sistem yang bisa memindahkan uang, bukan seperti demo chatbot.

