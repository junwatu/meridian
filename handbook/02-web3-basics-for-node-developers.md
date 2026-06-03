# 02. Web3 Basics for Node Developers

Bab ini menjelaskan istilah Web3 yang muncul di repo dengan bahasa developer aplikasi.

## Wallet

Wallet adalah identitas sekaligus rekening.

Di aplikasi biasa:

```text
user_id + auth token + balance di database
```

Di Web3:

```text
public key + private key + balance on-chain
```

`WALLET_PRIVATE_KEY` di `.env` adalah secret paling sensitif. Jika bocor, aset bisa hilang.

## Public Key dan Private Key

Public key seperti nomor rekening. Aman untuk dibagikan.

Private key seperti password root untuk rekening. Jangan pernah masuk Git.

Repo ini membaca private key lewat env, lalu membuat `Keypair` untuk signing transaction.

## Solana RPC

RPC adalah endpoint untuk membaca dan menulis data ke Solana.

Analogi backend:

```text
Solana RPC = database/API server untuk blockchain
```

Bedanya, write operation berupa transaksi yang harus ditandatangani wallet.

## Transaction

Transaction adalah request perubahan state di blockchain.

Contoh:

- Membuka posisi.
- Menutup posisi.
- Claim fee.
- Swap token.

Di Node.js backend, ini mirip write request ke database. Tapi setelah confirmed, transaksi sulit dibatalkan.

## Gas / Fee

Solana memakai transaction fee. Repo sering menyebut `gasReserve`, walaupun Solana tidak memakai istilah gas seperti Ethereum.

Mental model:

```text
gasReserve = saldo operasional agar agent masih bisa membayar biaya transaksi berikutnya
```

## Token

Token adalah aset digital. Bisa dianggap seperti item balance dengan mint address sebagai ID unik.

Contoh:

```text
SOL = native token Solana
USDC = stablecoin
TOKEN_X = token lain yang diperdagangkan di pool
```

## Pool

Pool adalah tempat dua token bisa ditukar.

Analogi:

```text
Pool = marketplace pasangan barang
SOL/USDC pool = marketplace untuk tukar SOL dan USDC
```

## Liquidity Provider

Liquidity provider adalah orang atau program yang menaruh modal ke pool agar trading bisa terjadi.

Sebagai imbalan, liquidity provider mendapat fee.

## DLMM

DLMM adalah model liquidity dari Meteora. Dalam repo ini, konsep pentingnya adalah `bin`.

## Bin

Bin adalah kotak harga kecil.

Analogi:

```text
Rak harga 100-101
Rak harga 101-102
Rak harga 102-103
```

`active_bin` adalah kotak harga saat ini.

`bins_below` adalah jumlah kotak di bawah harga saat ini yang dicakup posisi.

## Out of Range

Position out of range berarti harga bergerak keluar dari area yang dicakup position.

Analogi:

```text
Lapak kamu buka di lorong A, tapi semua pembeli pindah ke lorong B.
```

Posisi masih ada, tapi tidak efektif menghasilkan fee.

## RPC/API Failure Mode

Di Web3, kegagalan bisa datang dari banyak tempat:

- RPC lambat.
- API return data stale.
- Transaction simulation gagal.
- Transaction confirmed terlalu lama.
- Wallet saldo berubah.
- Pool state berubah di antara read dan write.

Karena itu repo ini banyak memakai fresh check sebelum action penting.

## Ringkasan

Untuk developer Node.js, Web3 bisa dipahami sebagai external state machine dengan write operation yang mahal dan irreversible.

Mental model:

```text
Blockchain = shared database
Wallet = account + signer
Transaction = signed write request
RPC = database gateway
```

