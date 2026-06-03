# 11. Outputs, Logs, and Learning

Meridian menghasilkan lebih dari sekadar transaksi.

## Outputs Utama

Sistem bisa menghasilkan:

- screening report
- management report
- open position
- claimed fees
- close result
- swap result
- Telegram alert
- decision log
- lessons
- pool memory
- runtime logs

## Logs

`logger.js` menulis log harian dan action audit.

Log penting untuk:

- debugging API failure
- mengetahui tool apa yang dipanggil
- melihat durasi tool
- melihat blocked action
- audit tindakan agent

## Decision Log

`decision-log.json` menyimpan alasan keputusan.

Entry bisa berisi:

- actor
- type
- pool
- position
- summary
- reason
- risks
- metrics
- rejected alternatives

Ini penting untuk explainability.

## Lessons

`lessons.js` menyimpan performance posisi yang sudah ditutup.

Sistem menghitung:

- PnL USD
- PnL percentage
- range efficiency
- fees earned
- close reason

Jika hasil sangat baik atau buruk, sistem membuat lesson.

## Pool Memory

`pool-memory.json` menyimpan history per pool.

Contoh:

- pool ini pernah profit
- pool ini sering out of range
- pool ini sedang cooldown
- adjusted win rate rendah

## Telegram Reports

Telegram memberi feedback operator:

- deploy notification
- close notification
- swap notification
- out-of-range warning
- live message saat cycle berjalan

## What Counts As Truth

Urutan source of truth:

1. Tool result.
2. On-chain/API state.
3. Local state/logs.
4. LLM text.

LLM text sendiri bukan bukti.

## Ringkasan

Sistem ini menghasilkan action dan memory. Memory membuat agent lebih mudah diaudit dan lebih konsisten dari waktu ke waktu.

Mental model:

```text
Every action should leave a trail.
```

