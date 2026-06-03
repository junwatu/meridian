# 10. Screening, Deploy, and Management

Bab ini menjelaskan tiga workflow inti.

## Screening

Screening mencari pool yang layak.

Input:

- pool data
- token data
- wallet state
- config threshold
- blacklist
- pool memory
- smart wallet signals

Output:

- candidate list
- skip reason
- deploy decision
- no-deploy report

## Candidate Scoring

Candidate dinilai dari sinyal seperti:

- fee/active-TVL
- organic score
- volume
- holders

Scoring bukan satu-satunya keputusan. Hard filter tetap lebih penting.

## Deploy

Deploy membuka liquidity position.

Sebelum deploy:

```text
LLM selects candidate
  -> executor validates
  -> dlmm tool computes range
  -> transaction is built
  -> dry-run or send real transaction
  -> state tracks position
```

## Bins

DLMM memakai bin sebagai area harga.

Agent memakai:

- `bins_below`
- `bins_above`
- `active_bin`

Untuk default single-side SOL deploy:

```text
bins_above = 0
upper bin = active bin
lower bin = active bin - bins_below
```

## Management

Management mengawasi posisi terbuka.

Possible action:

- `STAY`
- `CLAIM`
- `CLOSE`
- `INSTRUCTION`

Beberapa rule deterministic dijalankan tanpa LLM. LLM dipakai saat butuh judgment.

## Claim Fees

Claim mengambil fee yang sudah terkumpul.

Agent hanya claim jika jumlah cukup untuk membenarkan biaya transaksi.

## Close Position

Close mengakhiri posisi.

Setelah close, executor bisa auto-swap base token kembali ke SOL jika nilainya cukup.

## Out of Range

Jika posisi keluar range terlalu lama, management bisa close.

Config terkait:

- `outOfRangeWaitMinutes`
- `outOfRangeBinsToClose`
- `oorCooldownTriggerCount`
- `oorCooldownHours`

## Trailing Take Profit

Trailing take profit menyimpan peak PnL.

Jika PnL turun dari peak lebih dari threshold, agent bisa close untuk mengamankan profit.

## Ringkasan

Screening memilih peluang. Deploy membuka posisi. Management menjaga posisi agar tidak dibiarkan memburuk.

Mental model:

```text
Screening = should we enter?
Deploy = enter safely.
Management = should we stay or exit?
```

