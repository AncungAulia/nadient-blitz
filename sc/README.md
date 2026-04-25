# Nadient Smart Contracts

> **"Trust your eyes, outplay the odds."**

Smart contracts untuk Nadient — game Web3 skill-based wagering di Monad Testnet. Pemain bertaruh MockUSDC untuk mencocokkan warna secara visual dengan akurasi setinggi mungkin.

## Architecture

```
┌─────────────┐     ┌──────────────┐
│  MockUSDC   │────▶│ NadientGame  │
│  (ERC-20)   │     │              │
│             │     │ • Stake Lock │
│ • Faucet    │     │ • ECDSA Sig  │
│ • 6 decimal │     │ • Resolve    │
│ • 24h CD    │     │ • Withdraw   │
└─────────────┘     │ • Refund     │
                    │ • Reserve    │
                    └──────────────┘
```

| Contract | Purpose |
|----------|---------|
| **MockUSDC** | Custom ERC-20 dengan faucet (100 mUSDC per 24 jam). 6 decimals. |
| **NadientGame** | Core game contract — stake locking, ECDSA-verified round resolution, pull-pattern withdrawals, solo reserve pool, emergency pause. |

## Game Modes & Stakes

| Mode | Stake | Max Players | Payout |
|------|-------|-------------|--------|
| Solo | 5 mUSDC | 1 | From Solo Reserve Pool (tier-based) |
| Duel (1v1) | 10 mUSDC | 2 | 80% winner, 10% dev, 10% reserve |
| Battle Royale | 10 mUSDC | 5 | 80% winner, 10% dev, 10% reserve |

## Quick Start

### Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Git

### Install

```bash
git clone <repo-url>
cd monas-sc
forge install
```

### Build

```bash
forge build
```

### Test

```bash
forge test -vvv
```

### Deploy

```bash
cp .env.example .env
# Edit .env with your values
source .env
forge script script/Deploy.s.sol:Deploy --rpc-url $RPC_URL --broadcast
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `PRIVATE_KEY` | Deployer private key |
| `SIGNER_ADDRESS` | ECDSA signer address for score verification |
| `DEV_TREASURY` | Address to receive dev rake (10%) |
| `BACKEND_SIGNER` | Backend EOA for `resolveRound` and `refundStake` |
| `RPC_URL` | Monad Testnet RPC URL |

## Security Features

- **ECDSA Signature Verification** — All round results are signed off-chain and verified on-chain
- **Deadline Expiry** — Signatures expire to prevent stale replays
- **Payout Validation** — On-chain check: total payout ≤ total staked per round
- **Backend-Only Access** — `resolveRound` and `refundStake` restricted to backend signer
- **ReentrancyGuard** — All state-changing functions protected
- **Pull Pattern Withdrawals** — Anti-DoS; players claim via `withdraw()`
- **Emergency Pause** — Owner can pause deposits/resolves; withdrawals always available
- **Max Players Cap** — 5 players per round to prevent DoS via unbounded loops
- **Zero-Address Guards** — All setters and constructor validate non-zero addresses
- **SafeERC20** — Safe token transfer wrappers

## Test Coverage

27 tests covering:
- Full lifecycle flows (Duel, Solo Jackpot/Lose, Battle Royale 5-player)
- Security guards (signature, deadline, access control, payout limits, pause)
- Edge cases (double resolve, deposit to resolved/refunded round, round full)
- Admin functions (treasury migration, reserve drain, zero-address guards)

## License

MIT