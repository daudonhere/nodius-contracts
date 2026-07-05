# Nodius Contracts

Smart contracts for Nodius Wallet — gas abstraction relay across EVM, Solana, and TON.

## Repo Structure

```
nodius-contracts/
├── evm-contract/     NodiusRelay — EIP-712 meta-tx relay (Solidity, Hardhat)
├── sol-contract/     NodiusRelay — ed25519 relay + CPI execution (Anchor/Rust)
└── ton-contract/     TonGaslessWallet — external message gasless wallet (Tact)
```

## EVM Contract — `evm-contract/`

**NodiusRelay** — EIP-712 meta-transaction relay contract deployed on Sepolia and Base Sepolia.

### Environment

Copy `evm-contract/.env.example` to `evm-contract/.env`:

| Variable | Description |
|----------|-------------|
| `DEPLOYER_PRIVATE_KEY` | Deployer private key (required) |
| `ALCHEMY_API_KEY` | Alchemy API key for Sepolia RPC |
| `NETWORK` | Target network — `sepolia` or `base-sepolia` (default: `sepolia`) |

### Deployed Addresses

| Chain | Chain ID | Address |
|-------|----------|---------|
| Sepolia | `11155111` | `0x67f36e0c0bac9c2c92f81e94ec2cd1af07e06ae8` |
| Base Sepolia | `84532` | `0x67f36e0c0bac9c2c92f81e94ec2cd1af07e06ae8` |

### Commands

```bash
cd evm-contract
npm install
npm run compile                           # Compile Solidity
npm run deploy:sepolia                    # Deploy to Sepolia
npm run deploy:base-sepolia               # Deploy to Base Sepolia
```

### Flow

1. User signs an EIP-712 typed data (`Execute`) with their wallet
2. User sends signature + params to backend
3. Backend calls `NodiusRelay.execute()` — pays gas via relayer
4. Contract verifies EIP-712 signature, increments nonce, forwards call

---

## Solana Contract — `sol-contract/`

**NodiusRelay** — Anchor program for relay execution + fee sponsorship on Solana.

### Deployed

| Item | Value |
|------|-------|
| Program ID | `Gn5qgu9TVMZiQXxspugbzVKEJZRx7fEiGCw8c472Rq18` |
| Relayer | `9ErX5EiqVtr9Hr9G4y3kiJxm7xvXUL1dLjrmnXQgaUq1` |
| Cluster | devnet |

### Commands

```bash
cd sol-contract
anchor build                             # Build program
anchor deploy --provider.cluster devnet  # Deploy to devnet
anchor test                              # Run tests
```

### Instructions

| Instruction | Description |
|-------------|-------------|
| `initializeUser` | Create PDA nonce tracker for user |
| `relayExecute` | Verify Ed25519 signature → check nonce & deadline → CPI to target |

### Flow

1. User signs message (nonce, deadline, target, data) with ed25519
2. User sends signature + params to backend
3. Backend builds tx: relayer as fee_payer, `relayExecute` instruction
4. Program verifies signature via ed25519 precompile, executes CPI
5. Nonce incremented — replay protection

---

## TON Contract — `ton-contract/`

**TonGaslessWallet** — Tact-based gasless wallet that accepts external messages with signature verification.

### Environment

Copy `ton-contract/.env.example` to `ton-contract/.env`:

| Variable | Description |
|----------|-------------|
| `TON_DEPLOYER_MNEMONIC` | Deployer wallet 24-word mnemonic (required) |
| `TONCENTER_API_KEY` | TonCenter API key |
| `TON_RELAYER_MNEMONIC` | Fallback mnemonic if `TON_DEPLOYER_MNEMONIC` not set |

### Deployed

| Item | Value |
|------|-------|
| Contract | `EQAQqz3kY030tMSqoT6Ai588Ih67sEaecGZ64yEjdX5S63Rd` |
| Deployer | `EQCUhWYp6TZ_hAo6hoQa32U86ktKRepuEt9HXDU7hnv78wip` |
| Network | testnet |

### Commands

```bash
cd ton-contract
npm install
npm run compile                           # Compile Tact → build/
npm run deploy                            # Deploy to testnet
```

### Flow

1. User signs a `SignedExecute` message cell with their TON wallet
2. Backend sends external message to `TonGaslessWallet` contract
3. Contract verifies signature against stored owner public key
4. Contract forwards internal message to target with specified value
5. Seqno incremented — replay protection

### Status

TON gas abstraction is **partial**. Current deployed contract owner = deployer/relayer key.
Per-user smart wallet flow is required for full production use.

---

## License

MIT
