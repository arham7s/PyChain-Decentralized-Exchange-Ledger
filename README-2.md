# PyChain — Decentralized Exchange Ledger

> A production-grade blockchain simulator built entirely in the browser — no server, no build step, no dependencies beyond Chart.js.

![PyChain Banner](https://img.shields.io/badge/PyChain-v3.0-d4a017?style=for-the-badge&labelColor=080b10)
![Tech](https://img.shields.io/badge/ECDSA-P--256-00d4aa?style=for-the-badge&labelColor=080b10)
![Hash](https://img.shields.io/badge/SHA--256-PoW-8b7cf8?style=for-the-badge&labelColor=080b10)
![Deploy](https://img.shields.io/badge/GitHub%20Pages-Single%20File-d4a017?style=for-the-badge&labelColor=080b10)

---

## Overview

PyChain is a browser-native blockchain ledger designed for commodity exchange settlement simulation — a zero-dependency single-file web app bringing real cryptographic primitives, live analytics, and exchange-grade risk rules to any browser tab.

It runs entirely client-side using the **Web Crypto API** — all keypairs, signing, and hashing happen natively in your browser with no data ever leaving your machine.

**Live demo:** `https://arham7s.github.io/pychain` *(after deploying)*

---

## Features

### Cryptographic Layer
- **ECDSA P-256 Keypairs** — generated in-browser via `crypto.subtle.generateKey`; private keys never leave the session
- **Digital Signatures** — every transaction is signed with the sender's private key using `crypto.subtle.sign` before entering the mempool
- **Live Signature Verification** — the Block Inspector verifies each transaction's ECDSA signature on-chain using `crypto.subtle.verify`; tampered transactions show `SIG INVALID`
- **SHA-256 Proof-of-Work** — block hash computed as `SHA-256(merkleRoot + creatorId + timestamp + prevHash + nonce)`; difficulty 1–4 configurable via slider

### Chain Mechanics
- **Merkle Tree** — binary hash tree of transaction hashes computed before each mining run; stored per block and validated independently of the block hash
- **Mempool** — signed transactions queue before mining; Mine Block confirms all pending transactions at once
- **Chain Validation** — three independent checks per block: hash integrity, `prevHash` linkage, and Merkle root recomputation

### Instruments
| Symbol | Name |
|---|---|
| `CASH` | Cash Transfer |
| `NIFTY_FUT` | NIFTY 50 Futures |
| `BNKFUT` | BANKNIFTY Futures |
| `MCX_GOLD` | MCX Gold |
| `MCX_CRUDE` | MCX Crude Oil |
| `NCDEX_WHEAT` | NCDEX Wheat Futures |
| `USDINR` | USDINR Futures |

### Smart Contract Risk Rules
| Rule | Trigger | Behaviour |
|---|---|---|
| **Balance Guard** | Mempool entry | Rejects any transaction where sender balance < amount |
| **Margin Call Trigger** | Post-mining | Auto-injects `MARGIN_CALL` alert into mempool for any wallet below threshold |
| **Block Volume Alert** | Post-mining | Flags blocks exceeding concentration risk threshold in ledger + analytics |

All rules are configurable and togglable in the Risk Rules tab.

### Analytics Dashboard (Chart.js 4)
Six live charts update every time a block is mined:

1. **Cumulative Settlement Volume** — line chart of running total across blocks
2. **Instrument Breakdown** — doughnut by transaction count
3. **Mining Effort (Nonce) per Block** — bar chart; purple for high-effort blocks (>10k nonce)
4. **Block Volume vs Concentration Threshold** — bars with dashed risk threshold overlay
5. **Buy / Sell / Flow Pressure** — polar area chart of trade direction distribution
6. **Wallet Balance Distribution** — horizontal bars, red if below margin threshold

Plus four KPI cards: Peak Block Volume, Avg Mining Time, Top Instrument, Max Nonce Found.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Cryptography | Web Crypto API (`crypto.subtle`) — zero third-party crypto |
| Hashing | SHA-256 via `crypto.subtle.digest` |
| Signatures | ECDSA P-256 via `crypto.subtle.sign` / `crypto.subtle.verify` |
| Charts | [Chart.js 4.4.1](https://www.chartjs.org/) via cdnjs |
| Fonts | Syne · IBM Plex Mono · IBM Plex Sans via Google Fonts |
| Deployment | Single `index.html` — no build, no npm, no server |

---

## Project Structure

```
pychain/
├── index.html      ← entire application (HTML + CSS + JS, ~75KB)
└── README.md       ← this file
```

Everything lives in one file. CSS variables handle theming, vanilla JS handles state, and the Web Crypto API handles all cryptography. No React, no Webpack, no Node.

---

## Quickstart

### Option 1 — Open Locally
```bash
git clone https://github.com/arham7s/pychain.git
cd pychain
open index.html       # macOS
# or just double-click index.html in your file manager
```

### Option 2 — GitHub Pages
```bash
git clone https://github.com/arham7s/pychain.git
cd pychain
# Push to your repo, then:
# GitHub → Settings → Pages → Branch: main → / (root) → Save
# Live at: https://arham7s.github.io/pychain
```

No `npm install`. No `pip install`. Just open the file.

---

## How to Use

### Step 1 — Generate Wallets
Open the **Wallets** tab and click **Generate Wallet**. Each wallet gets a real ECDSA P-256 keypair. Create at least 2 (one sender, one receiver). Each starts with ₹10,00,000 initial balance.

### Step 2 — Sign a Transaction
Open the **Transaction** tab. Select From/To wallets, choose an instrument and side (BUY/SELL), enter an amount, then click **Sign & Add to Mempool**. The transaction is signed with the sender's private key before queuing.

### Step 3 — Mine a Block
In the **Mempool** panel, set difficulty (1–4) and click **⛏ Mine Block**. A Merkle root is computed for all pending transactions, then SHA-256 PoW runs until the hash target is met. Risk rules fire automatically post-mining.

### Step 4 — Inspect & Analyse
Click any block in the chain overview or ledger table to open the **Inspector** — which shows all transactions with live ECDSA signature verification. Click **Validate** to run full chain integrity checks. Scroll to **Analytics** to see all six charts.

---

## Block Data Schema

```js
Block {
  index:       number          // position in chain
  txs:         Transaction[]   // confirmed transactions
  merkleRoot:  string          // SHA-256 binary hash tree root
  creatorId:   number          // simulated miner ID (1–9)
  timestamp:   string          // ISO 8601
  prevHash:    string          // hash of previous block
  nonce:       number          // proof-of-work counter
  hash:        string          // SHA-256(merkleRoot+creatorId+timestamp+prevHash+nonce)
  miningTime:  number          // milliseconds
}

Transaction {
  id:          string          // unique tx identifier
  from:        string          // sender wallet address (0x + 16 hex chars)
  to:          string          // receiver wallet address
  instrument:  string          // CASH | NIFTY_FUT | BNKFUT | MCX_GOLD | ...
  side:        string          // BUY | SELL | TRANSFER
  qty:         number          // contracts / units (0 for cash)
  amount:      number          // ₹ value
  timestamp:   string          // ISO 8601
  signature:   string          // ECDSA P-256 hex signature of core tx fields
  pubHex:      string          // sender's raw public key in hex
}
```

---

## Wallet Address Derivation

```
address = "0x" + SHA-256(publicKeyHex).slice(0, 16)
```

Similar in principle to Ethereum's `keccak256(pubKey).slice(-20)` — deterministic, one-way, collision-resistant. The address cannot be reverse-engineered to reveal the private key.

---

## Proof-of-Work Difficulty Reference

| Difficulty | Expected Hashes | Typical Time |
|---|---|---|
| 1 | ~16 | Instant |
| 2 | ~256 | < 1 second |
| 3 | ~4,096 | 1–5 seconds |
| 4 | ~65,536 | 10–60 seconds |

---

## Chain Validation Logic

`Validate Chain` runs three independent checks on every block (genesis excluded):

```
1. SHA-256(block.merkleRoot + block.creatorId + block.timestamp
           + block.prevHash + block.nonce) === block.hash

2. block[i].prevHash === block[i-1].hash

3. recomputeMerkleRoot(block.txs) === block.merkleRoot
```

All three must pass for every block. Failure reports the specific block index and the type of mismatch.

---

## Possible Extensions

This codebase is designed to be extended. Some directions for a research paper or portfolio project:

- **SPAN/SPAN2 Margin Integration** — replace the flat threshold in Margin Call Trigger with a proper SPAN2 margin computation per wallet (scanning range × delta exposure)
- **Gossip Protocol Simulation** — use `BroadcastChannel` to simulate multiple browser tabs as nodes that propagate and validate blocks
- **VaR / CVaR on Block Volume Series** — compute rolling Value-at-Risk on the block settlement volume array visible in Analytics
- **Permissioned Viewing Keys** — add a Diffie-Hellman key exchange so only the intended receiver can decrypt transaction metadata (Hyperledger Fabric analogy)
- **Commodity Price Feed** — fetch live MCX gold or crude prices via a public API and auto-populate transaction amounts
- **Lamport Timestamps** — add distributed logical ordering to transactions for a more rigorous multi-node simulation

---

## License

MIT — use freely for academic, research, and portfolio purposes.

---

*Built with the Web Crypto API · Chart.js · Syne + IBM Plex · No build tools · No server · No dependencies*
