# 04. System Architecture

Arsitektur Meridian mengikuti pola Node.js service-oriented dengan LLM sebagai decision engine.

## Diagram Besar

```text
Scheduler / Telegram / CLI
        |
        v
    Agent Loop
        |
        v
 System Prompt + Live State + Lessons
        |
        v
      LLM
        |
        v
 Tool Call Request
        |
        v
 Executor Safety Layer
        |
        v
 Real Tool Implementation
        |
        v
 Chain/API/Local State
```

## Layer 1: Triggers

Trigger berasal dari:

- Cron scheduler.
- Telegram command.
- REPL terminal.
- CLI command.
- PnL poller.

Di aplikasi backend, ini mirip gabungan:

- HTTP controller
- queue worker
- cron job
- admin console

## Layer 2: Agent Loop

`agent.js` mengatur conversation dengan LLM.

Ia:

- membuat system prompt
- memilih tool yang boleh dipakai role
- mengirim request ke OpenRouter atau OpenAI-compatible server
- menjalankan tool call
- memasukkan hasil tool kembali ke LLM

## Layer 3: Context Builder

`prompt.js` membangun konteks:

- portfolio
- open positions
- state summary
- lessons
- performance
- recent decisions
- config

Ini mirip request context di backend, tapi lebih besar karena harus mengarahkan LLM.

## Layer 4: LLM

LLM bertugas melakukan judgment.

Contoh judgment:

- kandidat pool ini cukup kuat atau tidak
- posisi masih layak di-hold atau close
- risiko narrative/token terlalu tinggi atau tidak

LLM bukan source of truth. Tool result dan state file yang menjadi source of truth.

## Layer 5: Tool Request

LLM tidak memanggil function JavaScript langsung. Ia mengeluarkan structured tool call berdasarkan schema di `tools/definitions.js`.

Schema ini seperti OpenAPI spec internal untuk LLM.

## Layer 6: Executor Safety Layer

`tools/executor.js` menerima tool call, lalu:

- validasi tool name
- menjalankan safety checks
- dispatch ke implementation
- log action
- kirim notification
- update config/state jika perlu

Ini layer paling penting untuk keamanan.

## Layer 7: Tool Implementation

Tool implementation ada di:

- `tools/dlmm.js`
- `tools/screening.js`
- `tools/wallet.js`
- `tools/token.js`
- `tools/study.js`
- modul memory/config lain

Ini seperti service adapter untuk external systems.

## Layer 8: External and Local State

State yang disentuh:

- Solana chain
- Meteora APIs
- Helius
- Jupiter
- Telegram
- Agent Meridian API
- local JSON files

## Ringkasan

Meridian memakai arsitektur layered.

Mental model:

```text
LLM decides intent.
Executor enforces policy.
Tools perform action.
State records truth.
```

