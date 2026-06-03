# Meridian Core System Guide

Target pembaca: developer backend atau full-stack yang belum familiar dengan Web3.

Dokumen ini menjelaskan cara kerja inti Meridian dari sudut pandang software system. Fokusnya adalah alur program, modul, data, konfigurasi, dan batas keamanan. Istilah Web3 dijelaskan dengan analogi sederhana.

## 1. Gambaran Singkat

Meridian adalah agent otomatis untuk mengelola posisi liquidity provider di Meteora DLMM, sebuah sistem pool trading di Solana.

Kalau disederhanakan:

- Wallet = rekening operasional agent.
- Pool = marketplace kecil untuk dua token.
- Liquidity position = modal yang ditaruh agent di rentang harga tertentu.
- Fees = komisi kecil yang didapat ketika orang trading melewati rentang posisi.
- Out of range = harga bergerak keluar dari rentang posisi, sehingga posisi berhenti efektif menghasilkan fee.
- Agent = program Node.js yang membaca data, meminta LLM mengambil keputusan, lalu memanggil tool yang benar.

Mental model:

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

LLM tidak langsung menyentuh wallet. Semua tindakan harus melewati tool dan safety check di kode.

## 2. Entry Point Utama

### `index.js`

`index.js` adalah pusat runtime.

Tanggung jawabnya:

- Start aplikasi.
- Start cron job untuk screening dan management.
- Start Telegram polling.
- Start REPL terminal jika proses berjalan secara interaktif.
- Menjalankan daily briefing.
- Mengatur shutdown.
- Mencegah cycle yang sama berjalan paralel.

Cycle utama:

- `runScreeningCycle()` mencari pool baru dan deploy jika kandidat layak.
- `runManagementCycle()` memantau posisi yang sudah terbuka, lalu hold, claim fee, close, atau trigger screening baru.
- PnL poller 30 detik memantau exit condition ringan tanpa LLM.

### `cli.js`

`cli.js` adalah command-line wrapper.

Contoh command:

```bash
meridian balance
meridian positions
meridian candidates
meridian screen --dry-run
meridian manage --dry-run
meridian deploy --pool <pool> --amount <sol> --dry-run
meridian close --position <position> --dry-run
```

CLI berguna untuk debugging, automation, atau dipanggil oleh tool lain.

### `agent.js`

`agent.js` menjalankan ReAct loop.

ReAct berarti:

1. Agent membaca goal.
2. Agent melihat state dan prompt.
3. LLM memilih tool.
4. Tool dijalankan.
5. Hasil tool dikembalikan ke LLM.
6. Loop berulang sampai selesai atau mencapai batas step.

Ada tiga role:

| Role | Fungsi | Contoh tool |
|---|---|---|
| `SCREENER` | Cari pool dan deploy posisi baru | `get_top_candidates`, `deploy_position` |
| `MANAGER` | Kelola posisi terbuka | `get_my_positions`, `get_position_pnl`, `claim_fees`, `close_position` |
| `GENERAL` | Chat/manual command | Tool dipilih berdasarkan intent user |

Tool access dibatasi per role supaya screener tidak sembarang close posisi, dan manager tidak sembarang deploy.

## 3. Konfigurasi

### `config.js`

`config.js` membaca `user-config.json` dan environment variable.

Konfigurasi dibagi menjadi beberapa area:

- `risk`: batas jumlah posisi dan maksimal deploy.
- `screening`: filter pool sebelum LLM melihat kandidat.
- `management`: aturan close, claim, trailing take profit, gas reserve, ukuran posisi.
- `strategy`: strategi DLMM dan jumlah bin.
- `schedule`: interval management dan screening.
- `llm`: model, temperature, max token, max step.
- `api`: Agent Meridian relay dan public API.
- `hiveMind`: sinkronisasi lesson kolektif.
- `indicators`: konfirmasi chart indicator opsional.

Contoh konsep penting:

```js
computeDeployAmount(walletSol)
```

Fungsi ini menghitung jumlah SOL untuk deploy berdasarkan saldo wallet:

```text
deployable = walletSol - gasReserve
dynamic = deployable * positionSizePct
result = clamp(dynamic, deployAmountSol, maxDeployAmount)
```

Artinya posisi bisa membesar saat wallet membesar, tapi tetap punya batas bawah dan batas atas.

## 4. Prompt dan LLM

### `prompt.js`

`prompt.js` membangun system prompt sesuai role.

Prompt berisi:

- Portfolio saat ini.
- Posisi terbuka.
- Config aktif.
- Memory lokal.
- Lessons dari posisi sebelumnya.
- Recent decisions.
- Rule khusus untuk screener atau manager.

Poin penting:

- LLM diberi instruksi untuk tidak mengklaim aksi berhasil tanpa tool result.
- Data eksternal seperti token narrative, notes, dan memory dianggap untrusted.
- User instruction bisa override default strategy untuk manual action.
- Untuk action nyata, LLM wajib memanggil tool.

## 5. Tool Layer

### `tools/definitions.js`

File ini mendefinisikan schema tool yang dilihat LLM.

Schema menjelaskan:

- Nama tool.
- Deskripsi.
- Parameter.
- Required fields.
- Warning untuk action yang mengirim transaksi.

Ini mirip API contract antara LLM dan kode.

### `tools/executor.js`

Executor adalah gerbang utama sebelum tool dijalankan.

Tanggung jawabnya:

- Validasi nama tool.
- Jalankan safety check untuk protected tool.
- Dispatch ke function sebenarnya.
- Log action.
- Kirim Telegram notification jika ada deploy, close, claim, atau swap.
- Auto-swap base token ke SOL setelah close jika nilainya cukup.
- Persist perubahan config saat `update_config`.

Protected tools:

- `deploy_position`
- `claim_fees`
- `close_position`
- `swap_token`
- `self_update`

Untuk `deploy_position`, safety check penting meliputi:

- Pool harus lolos threshold fresh dari API.
- TVL harus dalam range config.
- Fee/active-TVL harus cukup.
- Volatility harus valid dan positif.
- Bin step harus dalam range config.
- Tidak boleh duplicate pool.
- Tidak boleh duplicate base token.
- Tidak boleh lebih dari `maxPositions`.
- Deploy harus single-side SOL, yaitu `amount_y` atau `amount_sol`, dan `amount_x=0`.
- Range minimal harus cukup lebar, bukan tiny range.
- Wallet harus punya SOL cukup untuk deploy plus gas reserve.

Kesimpulan: LLM boleh memberi saran, tapi executor yang memutuskan apakah action aman dijalankan.

## 6. Screening System

### `tools/screening.js`

Screening mencari pool kandidat dari Meteora Pool Discovery API dan sumber tambahan.

Alur sederhana:

```text
Fetch pools
  -> normalize metrics
  -> apply volatility timeframe
  -> hard filters
  -> enrich token data
  -> apply blacklist/cooldown/launchpad checks
  -> score candidates
  -> return top candidates
```

Filter utama:

- Market cap minimal dan maksimal.
- Holder count minimal.
- Volume minimal.
- Active TVL minimal dan maksimal.
- Bin step minimal dan maksimal.
- Fee/active-TVL minimal.
- Volatility harus positif.
- Organic score token base dan quote.
- Token warning dan ownership risk.
- Launchpad allow-list atau block-list.
- Cooldown pool atau base token.
- PVP symbol conflict.

Untuk developer non-Web3:

- `TVL` mirip total uang yang tersedia di marketplace.
- `volume` mirip traffic transaksi.
- `fee/TVL` mirip rasio pendapatan terhadap modal.
- `organic score` mirip skor apakah aktivitas terlihat natural.
- `holders` mirip jumlah pemilik token.
- `launchpad` mirip platform tempat token dibuat.

## 7. DLMM dan Wallet Layer

### `tools/dlmm.js`

File ini mengurus operasi Meteora DLMM.

Fungsi penting:

- `getMyPositions()`: membaca posisi wallet.
- `getWalletPositions()`: membaca posisi wallet mana pun.
- `getActiveBin()`: membaca bin harga aktif pool.
- `deployPosition()`: membuka posisi baru.
- `claimFees()`: mengambil fee dari posisi.
- `closePosition()`: menutup posisi.
- `getPositionPnl()`: membaca PnL dan fee metrics.
- `searchPools()`: mencari pool.

Istilah DLMM:

- `bin` = satu kotak harga kecil.
- `active bin` = kotak harga saat ini.
- `bins_below` = berapa kotak di bawah harga saat ini yang ingin dicakup.
- `bins_above` = berapa kotak di atas harga saat ini yang ingin dicakup.
- `bid_ask` dan `spot` = cara modal disebar di rentang harga.

Agent ini secara default memakai single-side SOL deploy:

```text
amount_y > 0
amount_x = 0
bins_above = 0
bins_below >= minimum safe range
```

Artinya agent menaruh SOL di bawah harga aktif agar mendapat posisi tertentu sesuai strategi.

### `tools/wallet.js`

File ini mengurus wallet balance dan swap.

Sumber data:

- Helius untuk balance wallet.
- Jupiter untuk swap token.

Jika `DRY_RUN=true`, swap tidak benar-benar dikirim.

## 8. State, Memory, dan Learning

### `state.js`

Menyimpan runtime state di `state.json`.

Data yang disimpan:

- Posisi yang pernah dibuka.
- Pool dan strategy.
- Bin range.
- Waktu deploy.
- Out-of-range timestamp.
- Fee claim history.
- Manual instruction untuk posisi.
- Peak PnL untuk trailing take profit.
- Recent events.

State ini melengkapi data on-chain. Blockchain bisa memberi posisi, tapi tidak selalu menyimpan konteks aplikasi seperti alasan deploy atau instruction user.

### `lessons.js`

Menyimpan lessons dan performance di `lessons.json`.

Saat posisi ditutup:

1. Sistem menghitung PnL.
2. Menghitung range efficiency.
3. Membuat lesson jika hasil sangat baik atau buruk.
4. Menyimpan performance.
5. Mengupdate pool memory.
6. Setiap beberapa close, mencoba evolve threshold.
7. Opsional mengirim lesson/performance ke HiveMind.

Catatan teknis:

Ada tech debt di `evolveThresholds()`. Fungsi ini masih memakai key `maxVolatility` dan `minFeeTvlRatio`, sedangkan config aktif memakai `minFeeActiveTvlRatio` dan tidak punya `maxVolatility`. Akibatnya sebagian evolution menjadi no-op atau tidak selaras dengan config.

### `pool-memory.js`

Menyimpan history per pool di `pool-memory.json`.

Fungsi utamanya:

- Mencatat hasil deploy yang sudah ditutup.
- Menghitung win rate.
- Menghitung adjusted win rate.
- Menyimpan notes.
- Memberi cooldown untuk pool atau token yang berulang kali bermasalah.

Ini seperti memory jangka panjang untuk pool tertentu.

### `decision-log.js`

Menyimpan alasan keputusan di `decision-log.json`.

Contoh keputusan:

- Deploy.
- Skip deploy.
- No deploy.
- Close.

Tujuannya agar agent bisa menjawab pertanyaan seperti:

```text
Why did you deploy?
Why did you skip?
Why did you close?
```

Tanpa decision log, agent mudah mengarang alasan setelah kejadian.

## 9. Telegram dan Operator Interface

### `telegram.js`

Telegram dipakai untuk:

- Mengirim notifikasi.
- Menerima command.
- Menampilkan live progress saat tool berjalan.
- Mengirim button untuk action tertentu.

Security penting:

- Bot tidak auto-register chat baru.
- `TELEGRAM_CHAT_ID` atau `user-config.telegramChatId` harus sudah ada.
- Untuk group chat, `TELEGRAM_ALLOWED_USER_IDS` wajib diisi.

Ini mencegah orang asing mengirim command ke bot.

## 10. Cron dan Runtime Flow

Saat `npm start` atau `node index.js`:

```text
load env
load config
bootstrap HiveMind
start background sync
fetch startup wallet/positions/candidates
start cron jobs
start Telegram polling
start REPL if TTY
```

Cron jobs:

- Management cycle setiap `managementIntervalMin`.
- Screening cycle setiap `screeningIntervalMin`.
- Health check setiap jam.
- Morning briefing setiap 1:00 UTC.
- Briefing watchdog setiap 6 jam.
- Lightweight PnL poller setiap 30 detik.

Race protection:

- `_managementBusy` mencegah dua management cycle overlap.
- `_screeningBusy` mencegah dua screening cycle overlap.
- `_screeningLastTriggered` mengurangi spam screening dari management.
- Deploy safety check tetap melakukan fresh position scan sebelum membuka posisi.

## 11. Typical Flows

### Flow: autonomous screening deploy

```text
cron triggers runScreeningCycle
  -> pre-check positions and balance
  -> compute deploy amount
  -> get top candidates
  -> enrich with smart wallets, token narrative, token info, pool memory
  -> filter risky candidates
  -> build SCREENER prompt
  -> LLM chooses deploy or skip
  -> executor validates deploy safety
  -> dlmm deploys or dry-runs
  -> state tracks position
  -> decision log records reason
  -> Telegram notifies operator
```

### Flow: autonomous management

```text
cron triggers runManagementCycle
  -> fetch live positions
  -> record pool snapshots
  -> update trailing TP state
  -> deterministic rule checks
  -> LLM handles cases needing judgment
  -> executor validates close/claim/swap
  -> dlmm or wallet tool executes
  -> lessons and pool memory update after close
  -> Telegram reports result
```

### Flow: manual Telegram or REPL command

```text
operator sends command
  -> index.js parses direct commands
  -> simple commands bypass LLM
  -> free-form commands go to GENERAL agent
  -> GENERAL role picks tool set by intent
  -> executor handles safety and logs result
```

## 12. Files That Matter Most

| File | Purpose |
|---|---|
| `index.js` | Main runtime, cron, REPL, Telegram handlers |
| `agent.js` | LLM loop and role-based tool filtering |
| `prompt.js` | System prompt builder |
| `config.js` | Runtime config and deploy amount logic |
| `tools/definitions.js` | Tool schema shown to LLM |
| `tools/executor.js` | Tool dispatch and safety checks |
| `tools/screening.js` | Pool discovery, filtering, scoring |
| `tools/dlmm.js` | Meteora position operations |
| `tools/wallet.js` | Wallet balances and Jupiter swap |
| `tools/token.js` | Token audit, holders, narrative |
| `state.js` | Local position state |
| `lessons.js` | Performance records and learned lessons |
| `pool-memory.js` | Per-pool deploy history and cooldowns |
| `decision-log.js` | Structured decision history |
| `telegram.js` | Telegram send/poll/control interface |
| `logger.js` | Logs and action audit trail |
| `cli.js` | CLI wrapper |

## 13. Runtime Files

These files are user-specific and gitignored:

| File | Meaning |
|---|---|
| `.env` | Secrets and env config |
| `user-config.json` | Local strategy/risk/runtime config |
| `state.json` | Open position state |
| `lessons.json` | Lessons and closed position performance |
| `pool-memory.json` | Pool history |
| `decision-log.json` | Recent decisions |
| `smart-wallets.json` | Watched wallets |
| `token-blacklist.json` | Blocked tokens |
| `dev-blocklist.json` | Blocked deployers/devs |
| `signal-weights.json` | Darwinian signal weights |
| `logs/` | Runtime logs |

Do not commit these files unless you are intentionally creating examples or fixtures.

## 14. External Services

| Service | Used for |
|---|---|
| Solana RPC | On-chain reads/writes |
| Meteora DLMM SDK | Position create/close/claim and pool operations |
| Meteora Pool Discovery API | Candidate pool discovery |
| Helius | Wallet balances |
| Jupiter | Token data and swaps |
| OpenRouter or compatible local LLM | Agent reasoning |
| Telegram Bot API | Operator notifications and commands |
| Agent Meridian API | HiveMind, relay, Discord signal candidates |
| OKX tooling | Token/smart-money/risk context |

## 15. How To Run Locally

Install:

```bash
npm install
```

Setup wizard:

```bash
npm run setup
```

Dry-run mode:

```bash
npm run dev
```

Live mode:

```bash
npm start
```

Syntax check:

```bash
npm test
```

Direct syntax command:

```bash
npm run test:syntax
```

PM2:

```bash
npm run pm2:start
npm run pm2:logs
npm run pm2:restart
```

## 16. Development Notes

When adding a new tool:

1. Add schema in `tools/definitions.js`.
2. Add implementation mapping in `tools/executor.js`.
3. Add role access in `agent.js`.
4. If it mutates on-chain state, add it to `WRITE_TOOLS`.
5. Add safety checks before execution if failure could lose money or change positions.
6. Add logging and decision context if the action affects strategy.

When changing screening:

1. Prefer hard filters in code before LLM sees candidates.
2. Keep LLM judgment for fuzzy context like narrative or conviction.
3. Make threshold names match `config.js`, `user-config.example.json`, and `update_config`.
4. Add examples to rejected/filtered output so no-deploy decisions are explainable.

When changing management:

1. Keep deterministic exit rules in code where possible.
2. Use LLM only when judgment is needed.
3. Keep close and swap behavior explicit.
4. Preserve `DRY_RUN=true` behavior.
5. Record performance after close.

## 17. Main Risks

- Live transaction risk: if `DRY_RUN=false`, deploy/claim/close/swap can send real transactions.
- Config drift: threshold names must match across config, docs, update tool, and lessons evolution.
- LLM hallucination: never trust text response alone. Only tool results prove action.
- External API failure: pool, wallet, token, and swap APIs can return stale or failed data.
- Runtime state corruption: malformed JSON in state files can make memory incomplete.
- Telegram control risk: chat ID and allowed user IDs must be configured carefully.
- Overlapping cycles: busy flags reduce risk, but new async flows should respect existing locks.

## 18. Quick Mental Model

Think of Meridian like an automated operations desk:

- Screening agent = analyst that finds opportunities.
- Manager agent = operator that watches active positions.
- Executor = compliance officer that blocks unsafe actions.
- DLMM/wallet tools = actual trading desk.
- State and lessons = notebook and performance journal.
- Telegram/CLI/REPL = control panel.

The LLM is not the system of record. The code, tool results, state files, and logs are.
