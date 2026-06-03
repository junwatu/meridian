# 03. Problems Solved

Meridian dibuat untuk mengatasi masalah operasional saat menjadi liquidity provider.

## Masalah 1: Terlalu Banyak Pool

Pool baru terus muncul. Tidak semua layak.

Developer analogy:

```text
Seperti ranking ribuan marketplace listing dengan sinyal noise tinggi.
```

Meridian memfilter pool berdasarkan:

- TVL
- volume
- fee/active-TVL
- holder count
- organic score
- market cap
- volatility
- launchpad
- token risk
- pool memory

## Masalah 2: Data Web3 Banyak Noise

Token bisa terlihat ramai padahal aktivitasnya tidak sehat.

Repo memakai beberapa sinyal:

- holder concentration
- bot holders
- bundler risk
- token narrative
- smart wallets
- blacklist
- blocked launchpads

Tidak semua sinyal sempurna, tapi kombinasi sinyal mengurangi blind deploy.

## Masalah 3: Posisi Perlu Dipantau 24/7

Liquidity position bisa berubah status kapan saja.

Risiko:

- out of range
- fee turun
- harga jatuh
- pool volume hilang
- posisi mencapai take profit

Meridian memakai management cron dan PnL poller untuk memantau posisi.

## Masalah 4: Keputusan Sering Tidak Terdokumentasi

Tanpa decision log, agent bisa menjawab pertanyaan dengan tebakan.

Repo menyimpan keputusan di `decision-log.json`:

- deploy
- skip
- no deploy
- close

Ini membuat tindakan lebih bisa diaudit.

## Masalah 5: LLM Bisa Hallucinate

LLM bisa menulis "deploy berhasil" tanpa benar-benar menjalankan transaksi.

Repo mengurangi risiko ini dengan:

- tool call wajib untuk action
- executor sebagai gerbang
- tool result sebagai bukti
- prompt anti-hallucination
- protected tools

## Masalah 6: Bot Bisa Terlalu Agresif

Tanpa risk config, bot bisa:

- deploy terlalu banyak
- memakai saldo terlalu besar
- deploy ke pool duplicate
- tidak menyisakan fee reserve
- masuk range terlalu sempit

Repo memakai risk config dan safety checks untuk mencegah ini.

## Masalah 7: Pengalaman Masa Lalu Sering Hilang

Jika pool pernah rugi, agent harus tahu.

Repo menyimpan:

- `lessons.json`
- `pool-memory.json`
- `state.json`
- `decision-log.json`

Memory ini dimasukkan kembali ke prompt.

## Ringkasan

Meridian bukan hanya script deploy. Ia mencoba menyelesaikan masalah operasi, risiko, observability, dan learning loop.

Mental model:

```text
Masalahnya bukan hanya "cara deploy".
Masalahnya adalah "kapan boleh deploy, kapan harus hold, dan kapan harus berhenti".
```

