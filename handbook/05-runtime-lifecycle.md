# 05. Runtime Lifecycle

Bab ini menjelaskan apa yang terjadi saat aplikasi berjalan.

## Startup

Saat `npm start` atau `node index.js`:

```text
load envcrypt
load config
init logger
bootstrap HiveMind
start background sync
fetch wallet/positions/candidates
start cron jobs
start Telegram polling
start REPL if interactive
```

Jika process non-TTY, aplikasi langsung start cron dan menjalankan screening cycle awal.

## Cron Jobs

`startCronJobs()` di `index.js` mengatur:

- management cycle
- screening cycle
- health check
- morning briefing
- briefing watchdog
- 30s PnL poller

## Screening Cycle

Screening mencari peluang baru.

Flow:

```text
pre-check balance and position count
  -> compute deploy amount
  -> fetch top candidates
  -> enrich candidates
  -> filter candidates
  -> ask SCREENER agent
  -> deploy or skip
  -> log decision
```

Jika sudah mencapai `maxPositions`, screening langsung skip.

## Management Cycle

Management mengelola posisi yang sudah terbuka.

Flow:

```text
fetch live positions
  -> update snapshots
  -> check deterministic exits
  -> decide hold/claim/close
  -> execute tools
  -> notify operator
```

Jika tidak ada posisi terbuka, management bisa trigger screening.

## PnL Poller

PnL poller berjalan setiap 30 detik.

Ia tidak memakai LLM. Tugasnya memantau kondisi cepat seperti:

- peak PnL
- trailing take profit
- deterministic close rule

Jika rule penting terpenuhi, poller dapat trigger management cycle.

## Telegram Runtime

Telegram menerima command dan callback button.

Command sederhana bisa langsung diproses.

Free-form message masuk ke GENERAL agent.

Security:

- chat ID harus cocok
- group command perlu allowed user IDs

## Shutdown

Shutdown harus:

- stop cron
- stop Telegram polling
- clear intervals/timers
- flush proses yang sedang jalan jika memungkinkan

## Race Protection

Ada beberapa flag:

- `_managementBusy`
- `_screeningBusy`
- `_screeningLastTriggered`
- `_pollTriggeredAt`

Tujuannya mencegah overlapping cycles dan double deploy.

## Ringkasan

Meridian berjalan seperti worker service yang punya beberapa trigger paralel, tapi memakai lock sederhana untuk menjaga action penting tidak overlap.

Mental model:

```text
Many triggers, one controlled execution path.
```

