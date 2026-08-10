# ProofX

ProofX is a privacy-focused decentralized application (DApp) built on the Midnight Network. It allows users to create, manage, and verify zero-knowledge proofs without exposing the underlying private information.

## Project Vision

ProofX enables users to prove statements about private data (age, education, membership, achievements, ownership, authorization) without revealing that data on-chain. By leveraging Midnight's zero-knowledge architecture, ProofX ensures that secrets never leave the user's device—only cryptographic commitments and nullifiers hit the blockchain. Verifiers learn *that* a statement is true, not *what* the secret is. This privacy-first approach is essential for identity verification, credential sharing, and access control in a world where data protection is paramount.

## Smart Contract Deployment

- **Network:** Preview (Testnet)
- **Deployed contract ID:** `[PENDING — run: npm run deploy -- --network preview]`
- **Contract source:** `contracts/proofx.compact`
- **Compiler:** Compact 0.5.1 (see `COMPILER_LIMITATIONS.md` for details)

> **Note:** The current implementation uses Compact 0.5.1 which has significant limitations (see below). Full ZK privacy with poseidon hashes, private witnesses, and Map/Set ledger state requires Compact 0.15+. The deployed contract discloses the secret as the commitment in this compatibility version.

## Key Features

- **Create Proofs:** Commit a secret to the blockchain as a cryptographic commitment with a derived nullifier
- **Nullifier Protection:** Each secret generates a unique nullifier (`secret + "-nullifier"`) preventing double-spending/replay
- **Privacy-Preserving:** In this compat version, the secret is disclosed as commitment; full ZK privacy (poseidon hashes, private witnesses) requires Compact 0.15+
- **Wallet Integration:** Connects to Lace wallet via Midnight DApp Connector on Preview testnet
- **Real-time State:** Reads contract state (commitment, nullifier) via Midnight indexer
- **Multi-proof Types:** Support for age, education, membership, achievement, ownership, authorization, and custom proofs
- **Transaction History:** Tracks proof creation activity with transaction IDs

## Future Scope

- **Full ZK Privacy:** Upgrade to Compact 0.15+ for poseidon hashing, private witnesses, and proper nullifier derivation (`poseidon(secret || "nullifier")`)
- **VerifyProof Circuit:** On-chain ZK proof verification against stored commitments
- **RevokeProof Circuit:** Owner-only proof revocation by proving knowledge of secret
- **Map/Set Ledger State:** Replace string-based storage with proper `Map<Bytes<32>, ProofRecord>` and `Set<Bytes<32>>` for nullifier tracking
- **Selective Disclosure:** Allow revealing subsets of secret data
- **Proof Aggregation:** Batch multiple proofs for efficiency
- **Mainnet Deployment:** Path to Midnight mainnet when available

## Tech Stack

| Layer | Technology |
|-------|------------|
| Smart Contract | Compact 0.5.1 (Midnight) |
| Blockchain | Midnight Network (Preview Testnet) |
| Backend | Node.js 22, TypeScript, Midnight.js 4.1.1, Wallet SDK 1.2.0 |
| Frontend | React 18, TypeScript, Vite, Tailwind CSS, Zustand |
| Wallet | Lace Wallet (DApp Connector API) |
| Indexer | Midnight Indexer (GraphQL) |
| Proof Server | Midnight Proof Server |
| Local Devnet | Docker Compose (Node, Indexer, Proof Server) |
| Testing | Vitest |
| Deployment | Vercel / Netlify ready |

## Local Development

### Prerequisites

- **Node.js 22+** (LTS recommended)
- **Docker Desktop** with WSL2 backend (Windows) or Docker Engine (Linux/macOS)
- **Compact 0.5.1** compiler (install via `cargo install --locked midnight-compact` or download binary)
- **Lace Wallet** browser extension for Preview testnet interaction

### Quick Start

```bash
# 1. Install dependencies
npm install
cd frontend && npm install && cd ..

# 2. Compile the contract
npm run compile

# 3. Start local devnet (optional - for local testing)
npm run proof-server:start
docker compose up -d --wait

# 4. Deploy to local devnet
npm run setup

# 5. Run tests
npm test

# 6. Start frontend development server
npm run frontend:dev
```

### Deploy to Preview Testnet

```bash
# 1. Ensure you have tNIGHT on Preview testnet
# Visit: https://faucet.preview.midnight.network
# Request tNIGHT for your Lace wallet address

# 2. Deploy to Preview
npm run deploy -- --network preview

# 3. Contract address will be printed and saved to .midnight-state.json
# Copy it to frontend/.env.local as VITE_CONTRACT_ADDRESS
```

### Available Scripts

| Script | Description |
|--------|-------------|
| `npm run compile` | Compile the ProofX contract (`contracts/proofx.compact` → `contracts/managed/proofx/`) |
| `npm run setup` | One-shot: start devnet, compile, deploy to local devnet |
| `npm run deploy` | Deploy compiled contract (use `--network preview` for Preview testnet) |
| `npm run cli` | Interactive CLI to create/read proofs on deployed contract |
| `npm run check-balance` | Check wallet NIGHT and DUST balances |
| `npm run test` | Run Vitest unit tests (9 tests) |
| `npm run test:e2e` | End-to-end smoke test against deployed contract |
| `npm run frontend:dev` | Start Vite dev server for frontend |
| `npm run frontend:build` | Build frontend for production |
| `npm run proof-server:start` | Start local proof server via Docker |
| `npm run proof-server:stop` | Stop local proof server |
| `npm run clean` | Remove compiled artifacts and wallet state |

### Network Configuration

The project supports three networks (configured in `src/network.ts`):

| Network | Node | Indexer | Proof Server | Faucet |
|---------|------|---------|--------------|--------|
| `undeployed` (local) | ws://127.0.0.1:9944 | http://127.0.0.1:8088 | http://127.0.0.1:6300 | N/A (genesis funded) |
| `preview` | https://rpc.preview.midnight.network | https://indexer.preview.midnight.network | https://proof-server.preview.midnight.network | https://faucet.preview.midnight.network |
| `preprod` | https://rpc.preprod.midnight.network | https://indexer.preprod.midnight.network | https://proof-server.preprod.midnight.network | https://faucet.preprod.midnight.network |

Switch networks: `npm run network preview` or `npm run deploy -- --network preview`

### Frontend Configuration

Create `frontend/.env.local` from `frontend/.env.example`:

```env
VITE_NETWORK=preview
VITE_INDEXER_URL=https://indexer.preview.midnight.network/api/v4/graphql
VITE_INDEXER_WS_URL=wss://indexer.preview.midnight.network/api/v4/graphql/ws
VITE_PROOF_SERVER_URL=https://proof-server.preview.midnight.network
VITE_CONTRACT_ADDRESS=your_deployed_contract_address_here
VITE_PRIVATE_STATE_PASSWORD=Local-Devnet-Development-Placeholder-1
```

### Project Structure

```
proofx/
├── contracts/
│   ├── proofx.compact           # Compact source (compat version)
│   └── managed/proofx/          # Compiled artifacts (gitignored)
├── scripts/
│   └── e2e-check.ts             # End-to-end smoke test
├── src/
│   ├── network.ts               # Network config + state persistence
│   ├── wallet.ts                # Wallet construction + sync cache
│   ├── setup.ts                 # Setup orchestrator
│   ├── deploy.ts                # Contract deployment
│   ├── cli.ts                   # Interactive CLI
│   └── check-balance.ts         # Balance checker
├── tests/
│   └── proofx.test.ts           # Vitest tests (9 tests)
├── frontend/
│   ├── src/
│   │   ├── pages/               # Dashboard, CreateProof, VerifyProof, MyProofs, Activity
│   │   ├── components/          # UI components, Proof components, Layout
│   │   ├── hooks/               # useAppStore (Zustand), useWallet
│   │   ├── services/midnight/   # Wallet & Contract services
│   │   ├── types/               # TypeScript types
│   │   └── utils/               # Helpers
│   ├── .env.example             # Environment template
│   ├── vercel.json              # Vercel deployment config
│   ├── netlify.toml             # Netlify deployment config
│   └── public/_redirects        # SPA routing
├── docker-compose.yml           # Local devnet (node, indexer, proof-server)
├── package.json                 # Root scripts + dependencies
├── vercel.json                  # Root Vercel config
├── netlify.toml                 # Root Netlify config
├── tsconfig.json                # TypeScript config
├── COMPILER_LIMITATIONS.md      # Compact 0.5.1 limitations doc
└── README.md                    # This file
```

## Compiler Limitations (Compact 0.5.1)

The current ProofX implementation uses Compact 0.5.1 which has severe limitations:

1. **Only `Opaque<"string">` ledger types supported** — No `Bytes<32>`, `Map`, `Set`, `Field`, or `Uint64`
2. **Single circuit limit** — Only the first circuit works reliably; subsequent circuits fail on variable assignments, `require`, and stdlib functions
3. **No `poseidon` hash** — Cannot implement proper commitment/nullifier derivation
4. **No Map/Set ledger state** — Cannot store multiple proofs or nullifier sets
4. **No private witnesses** — All circuit inputs are effectively public

**Impact:** The current contract discloses the secret directly as the commitment. Full ZK privacy requires upgrading to Compact 0.15+.

See `COMPILER_LIMITATIONS.md` for detailed analysis.

## Testing

```bash
# Run unit tests (9 tests covering circuit logic, state transitions, privacy)
npm test

# Run e2e test against deployed contract
npm run test:e2e
```

Tests cover:
- Circuit logic (nullifier derivation, commitment disclosure)
- State transitions (ledger updates, overwrites)
- Privacy guarantees (secret handling, disclosure tracking)
- Nullifier properties (uniqueness, determinism)
- Contract structure validation

## Deployment

### Vercel
1. Connect repository to Vercel
2. Set environment variables from `frontend/.env.example`
3. Build command: `npm run frontend:build`
4. Output directory: `frontend/dist`

### Netlify
1. Connect repository to Netlify
2. Set environment variables from `frontend/.env.example`
3. Build command: `npm run frontend:build`
4. Publish directory: `frontend/dist`

### Docker (Full Stack)
```bash
docker compose up -d --build
```

## Privacy Story

**Public (on-chain):** Commitment (secret), Nullifier (secret + "-nullifier"), Transaction metadata
**Private (never on-chain):** The original secret value, all form data entered by user
**Disclosed:** Secret as commitment, derived nullifier — by design in this compat version

**Full ZK Privacy (Compact 0.15+):** Only `poseidon(secret)` and `poseidon(secret || "nullifier")` would be on-chain. The secret never leaves user's device.

## License

MIT
