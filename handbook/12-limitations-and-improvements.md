# 12. Limitations and Improvements

Bab ini menjelaskan batasan sistem dan ide improvement.

## Limitations

### Financial Risk

Repo ini bisa mengirim transaksi nyata. Bug, config salah, atau data buruk bisa menyebabkan rugi.

### External API Dependency

Sistem bergantung pada:

- Solana RPC
- Meteora APIs
- Helius
- Jupiter
- OpenRouter/local LLM
- Telegram
- Agent Meridian API

Jika API gagal atau stale, keputusan bisa terganggu.

### LLM Judgment Risk

LLM bisa salah menilai narrative atau risiko.

Safety checks mengurangi risiko action, tapi tidak membuat judgment sempurna.

### Local JSON Storage

JSON files sederhana dan mudah debug, tapi tidak ideal untuk:

- multi-instance
- concurrent writes berat
- analytics kompleks
- schema migration

### Config Drift

Config key harus konsisten di banyak tempat:

- `config.js`
- `user-config.example.json`
- `tools/executor.js`
- `lessons.js`
- docs

Saat ini ada known issue di `evolveThresholds()`.

### Small Capital Inefficiency

Real deploy dengan modal sangat kecil tidak efisien karena:

- transaction fee
- rent/account creation
- close/claim/swap cost
- range management

Dry-run lebih cocok untuk belajar.

## Improvement Ideas

### 1. Fix Threshold Evolution

Perbaiki `evolveThresholds()` agar memakai key config yang benar.

Prioritas:

- `minFeeTvlRatio` -> `minFeeActiveTvlRatio`
- hapus atau tambahkan config resmi untuk `maxVolatility`

### 2. Add Simulation Mode

Tambahkan simulator lokal:

```text
mock wallet
mock pool
mock price movement
mock fees
```

Ini akan membantu pemula belajar tanpa chain.

### 3. Add Safety Tests

Buat test untuk:

- duplicate pool
- insufficient balance
- invalid volatility
- too narrow range
- amount_x > 0
- maxPositions reached

### 4. Add Replay Mode

Replay dari logs/decision-log untuk memahami keputusan masa lalu.

### 5. Add Dashboard

Dashboard lokal bisa menampilkan:

- open positions
- PnL
- pool memory
- recent decisions
- lessons
- config

### 6. Stronger Schema Validation

Gunakan validator seperti Zod untuk config dan runtime JSON.

### 7. Better Integration Tests

Tambahkan fixture untuk candidate pool dan mocked API responses.

## Learning Path

Untuk pemula Web3 yang sudah Node.js:

1. Jalankan syntax test.
2. Baca `config.js`.
3. Baca `agent.js`.
4. Baca `tools/executor.js`.
5. Jalankan dry-run.
6. Baca output candidate.
7. Trace satu screening cycle.
8. Trace satu management cycle.
9. Baru pelajari `tools/dlmm.js`.

## Ringkasan

Meridian sudah punya struktur yang baik untuk agentic DeFi automation, tapi tetap butuh testing, simulation, dan config hygiene agar lebih aman dikembangkan.

Mental model:

```text
Improve observability before increasing autonomy.
```

