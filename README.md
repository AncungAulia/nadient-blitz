# Nadient Blitz

> **"Trust your eyes, outplay the odds."**

Nadient adalah game Web3 skill-based **color matching** berbasis wagering yang berjalan di **Monad Testnet**. Pemain bertaruh mUSDC, lalu mencocokkan warna target secara visual. Akurasi menentukan payout — bukan keberuntungan.

![Nadient Screenshot](/screenshots/1.png)

---

## Cara Bermain

1. **Connect wallet** ke Monad Testnet
2. **Klaim mUSDC** gratis dari faucet (100 mUSDC / 24 jam)
3. Pilih mode permainan:
   - **Practice** — latihan gratis, tanpa taruhan
   - **Solo** — bertaruh mUSDC, lawan color matching system
   - **Multiplayer** — buat atau join room, lawan teman (2–5 pemain)
4. Cocokkan warna target seakurat mungkin dalam waktu yang ditentukan
5. Klaim kemenangan di halaman **Vault**

### Tier Payout (Solo Mode)

| Tier | Akurasi | Reward |
|------|---------|--------|
| JACKPOT | ≥ 98% | 10.0 mUSDC |
| GREAT | ≥ 90% | 7.5 mUSDC |
| GOOD | ≥ 75% | 6.0 mUSDC |
| MISS | < 75% | 0 mUSDC |

---

## Tech Stack

### Frontend
- **Next.js 16** + **React 19** (App Router)
- **Wagmi v3** + **Viem** — wallet & onchain interaction
- **Supabase** — database (leaderboard, match history)
- **Upstash Redis** — real-time room state & matchmaking
- **Tailwind CSS v4** + **shadcn/ui** — UI components

### Smart Contracts
- **Solidity 0.8.24** + **Foundry**
- **NadientGame.sol** — stake locking, ECDSA-verified round resolution, pull-pattern withdrawal, solo reserve pool
- **MockUSDC.sol** — ERC-20 test token dengan faucet

### Infrastructure
- **Monad Testnet** (Chain ID: 10143)
- **ECDSA backend signer** — verifikasi hasil round secara off-chain lalu submit ke contract

---

## Arsitektur

```
┌─────────────────────────────────────────────────────┐
│                   Next.js App                       │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │  /play   │  │   /me    │  │     /vault       │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │              API Routes                      │   │
│  │  /rooms/*  /matchmaking/*  /play/*  /vault/* │   │
│  └──────────────────┬────────────────────────── ┘  │
└─────────────────────┼───────────────────────────────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Supabase    Upstash     Monad RPC
      (Postgres)  (Redis)   (via Viem)
                                  │
                                  ▼
                          NadientGame.sol
                          MockUSDC.sol
```

### Flow Solo Mode
1. Pemain stake mUSDC → `depositStake()` on-chain
2. Server generate `roundId` (bytes32)
3. Pemain submit jawaban → server hitung akurasi
4. Backend sign hasil → `resolveRound()` on-chain dengan ECDSA signature
5. Pemain claim dari balance via `withdraw()`

### Flow Multiplayer (Friend Room)
1. Leader buat room dengan kode 6 karakter
2. Pemain lain join via kode atau link `/play/lobby/[code]`
3. Semua pemain ready & stake → room otomatis start
4. Round berjalan serentak, hasil di-resolve on-chain
5. Winner claim dari Vault

---

## Smart Contracts

| Contract | Address (Monad Testnet) |
|----------|------------------------|
| NadientGame | `0x5f2d05d60523ace45fAfaFD417f74C53de3D076A` |
| MockUSDC | `0x44FF2171847768800FE0CDB059aBe32E3F8d88eC` |

---

## Setup & Jalankan Lokal

### Prerequisites
- Node.js 20+
- Foundry (untuk smart contract)
- Akun Supabase + Upstash Redis

### Frontend

```bash
cd fullstack
cp .env.example .env   # isi semua variabel
npm install
npm run dev
```

### Environment Variables

```env
SIGNER_PRIVATE_KEY=              # private key backend signer
BACKEND_PRIVATE_KEY=             # private key untuk operasi backend
SUPABASE_SERVICE_ROLE_KEY=       # supabase service role key
NEXT_PUBLIC_SUPABASE_URL=        # supabase project URL
UPSTASH_REDIS_REST_URL=          # upstash redis URL
UPSTASH_REDIS_REST_TOKEN=        # upstash redis token
NEXT_PUBLIC_GAME_ADDRESS=        # NadientGame contract address
NEXT_PUBLIC_USDC_ADDRESS=        # MockUSDC contract address
NEXT_PUBLIC_CHAIN_ID=10143       # Monad Testnet
NEXT_PUBLIC_RPC_URL=https://testnet-rpc.monad.xyz
```

### Smart Contracts

```bash
cd sc
forge install
forge test
forge script script/Deploy.s.sol --rpc-url https://testnet-rpc.monad.xyz --broadcast
```

---

## Tim

Dibuat untuk **Monad Blitz Jogja** hackathon.

---

## Lisensi

MIT
