# 09. Config and Runtime State

Meridian memakai kombinasi env vars, `user-config.json`, dan local JSON files.

## Environment Variables

Secrets ada di `.env`.

Contoh:

```env
WALLET_PRIVATE_KEY=
RPC_URL=
OPENROUTER_API_KEY=
HELIUS_API_KEY=
TELEGRAM_BOT_TOKEN=
TELEGRAM_CHAT_ID=
DRY_RUN=true
```

Jangan commit `.env`.

## user-config.json

`user-config.json` menyimpan config non-secret:

- risk limits
- screening thresholds
- management behavior
- schedule
- LLM model
- strategy

File ini juga gitignored karena bisa berisi RPC URL atau preferensi personal.

## Config Loader

`config.js`:

- membaca `user-config.json`
- apply env fallback
- expose `config`
- expose `computeDeployAmount()`
- expose `reloadScreeningThresholds()`

## Runtime JSON Files

| File | Fungsi |
|---|---|
| `state.json` | posisi dan event runtime |
| `lessons.json` | performance dan lessons |
| `pool-memory.json` | history per pool |
| `decision-log.json` | audit keputusan |
| `smart-wallets.json` | wallet yang dipantau |
| `token-blacklist.json` | token yang diblokir |

Ini seperti database lokal ringan.

## Kenapa JSON, Bukan Database

Keuntungannya:

- mudah dibaca
- mudah debug
- tidak perlu setup DB
- cocok untuk single-agent local runtime

Kekurangannya:

- tidak ideal untuk concurrency berat
- corrupt JSON bisa membuat data hilang
- query terbatas
- tidak cocok untuk multi-instance tanpa locking

## update_config Tool

`update_config` memungkinkan agent mengubah config runtime.

Ia:

- validasi key
- normalize value
- update object live
- persist ke `user-config.json`
- restart cron jika interval berubah
- menulis lesson untuk perubahan tertentu

## Config Drift

Masalah penting: nama config harus konsisten.

Contoh tech debt saat ini:

- `config.js` memakai `minFeeActiveTvlRatio`
- `lessons.js evolveThresholds()` masih memakai `minFeeTvlRatio`

Ini membuat sebagian evolution tidak efektif.

## Ringkasan

Config adalah policy runtime. State JSON adalah memory lokal. Keduanya harus diperlakukan sebagai bagian dari sistem, bukan file sampingan.

Mental model:

```text
.env = secrets
user-config.json = runtime policy
state JSON = local memory
logs = audit trail
```

