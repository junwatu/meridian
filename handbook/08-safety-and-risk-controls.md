# 08. Safety and Risk Controls

Bab ini penting karena repo bisa mengirim transaksi nyata.

## DRY_RUN

Gunakan:

```bash
DRY_RUN=true npm run dev
```

Dengan dry-run:

- agent tetap membaca data
- logic tetap berjalan
- deploy/swap tidak mengirim transaksi nyata

Dry-run adalah mode wajib untuk belajar.

## Protected Tools

Protected tools di `tools/executor.js`:

- `deploy_position`
- `claim_fees`
- `close_position`
- `swap_token`
- `self_update`

Tool ini melewati safety checks sebelum dijalankan.

## Deploy Safety Checks

Sebelum deploy:

- pool diverifikasi ulang dengan fresh data
- TVL harus sesuai config
- fee/active-TVL harus cukup
- volatility harus positif
- bin step harus sesuai range
- jumlah posisi belum mencapai `maxPositions`
- tidak boleh duplicate pool
- tidak boleh duplicate base token
- deploy harus single-side SOL
- range tidak boleh terlalu sempit
- wallet harus punya saldo cukup

## Risk Config

Config utama:

```js
risk: {
  maxPositions,
  maxDeployAmount,
}
```

Config terkait:

```js
management: {
  deployAmountSol,
  gasReserve,
  positionSizePct,
  minSolToOpen,
}
```

## Gas Reserve

`gasReserve` adalah saldo yang tidak boleh dipakai deploy.

Analogi:

```text
operational cash reserve
```

Tanpa reserve, agent mungkin bisa open position tapi tidak punya cukup SOL untuk close, claim, atau swap.

## Max Positions

`maxPositions` membatasi jumlah posisi terbuka.

Ini mencegah agent menyebar modal terlalu banyak dan membuat monitoring sulit.

## Single-Side SOL Rule

Agent ini dirancang untuk deploy memakai SOL side saja:

```text
amount_y > 0
amount_x = 0
bins_above = 0
```

Ini menyederhanakan risiko dan perhitungan.

## Telegram Security

Telegram tidak boleh dianggap aman secara default.

Repo mengharuskan:

- chat ID cocok
- group command butuh allowed user IDs

## Common Mistakes

- Menjalankan live mode tanpa memahami config.
- Menganggap dry-run sama dengan simulasi ekonomi lengkap.
- Menaruh private key di file yang bisa masuk Git.
- Menurunkan `gasReserve` terlalu kecil.
- Meningkatkan `maxPositions` tanpa monitoring.
- Mempercayai narrative token tanpa cross-check.

## Ringkasan

Safety di Meridian tidak bergantung pada LLM. Safety berada di kode.

Mental model:

```text
LLM can ask. Executor can refuse.
```

