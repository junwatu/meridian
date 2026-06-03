# 01. Orientation

Meridian adalah autonomous agent untuk mengelola liquidity position di Meteora DLMM pools di Solana.

Kalau kamu Node.js full-stack developer, pikirkan Meridian sebagai:

```text
background worker + AI decision engine + external API adapters + local state store + operator interface
```

Perbedaannya: beberapa action bisa mengirim transaksi finansial nyata ke blockchain.

## Apa Yang Dilakukan

Meridian melakukan empat pekerjaan besar:

1. Mencari pool yang terlihat layak.
2. Membuka liquidity position.
3. Memantau posisi yang sudah terbuka.
4. Menutup, claim fee, atau swap token saat kondisi tertentu terpenuhi.

Selain itu, Meridian menyimpan alasan keputusan dan belajar dari posisi yang sudah selesai.

## Bukan Sekadar Chatbot

LLM di repo ini bukan hanya menjawab pertanyaan. LLM menjadi decision layer.

Namun LLM tidak langsung memegang wallet. LLM hanya bisa meminta tool call. Tool call melewati executor dan safety checks.

Mental model:

```text
LLM = planner
executor = safety middleware
tools = service layer
wallet/RPC/API = external systems
```

## Apa Itu Liquidity Position

Liquidity position adalah modal yang ditaruh di pool agar orang lain bisa trading.

Analogi:

```text
Pool = pasar
Liquidity = stok barang
Position = lapak kamu di pasar
Fee = komisi dari transaksi pembeli
```

Di DLMM, position aktif dalam rentang harga tertentu. Jika harga keluar dari rentang itu, posisi disebut out of range dan biasanya tidak efektif menghasilkan fee.

## Kenapa Sistem Ini Perlu Agent

DeFi pool berubah cepat:

- Volume bisa naik turun.
- Harga bisa keluar range.
- Token bisa berisiko.
- Fee bisa menurun.
- Pool baru muncul terus.

Manual monitoring 24/7 sulit. Meridian mencoba mengotomasi monitoring dan keputusan operasional.

## Cara Repo Ini Berpikir

Repo ini menggabungkan tiga pendekatan:

1. Hard rules di kode untuk hal yang tidak boleh salah.
2. LLM judgment untuk konteks yang fuzzy.
3. Memory/lessons supaya keputusan berikutnya punya riwayat.

Contoh:

- Hard rule: jangan deploy jika wallet kurang saldo.
- LLM judgment: apakah narrative token cukup kuat?
- Memory: pool ini pernah rugi karena out of range berulang.

## Ringkasan

Meridian adalah sistem agentic finance. Ia memakai LLM untuk mengambil keputusan, tapi memakai kode sebagai pagar keamanan.

Mental model:

```text
AI boleh memberi instruksi, tapi kode yang memberi izin.
```

