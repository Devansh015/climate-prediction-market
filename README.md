# Klashi Climate Markets

Klashi is an experimental climate-risk prediction market interface built with Next.js and an Anchor program for Solana. It combines a geographic market explorer, clearly labelled sample climate markets, wallet-based Devnet transactions, and a simple pooled binary settlement model.

> [!WARNING]
> This repository is a Devnet prototype, not a production financial service. Every seeded market, chart, trade, position, and evidence summary is fictional **SAMPLE DATA**. Connected wallet balances come from Devnet, whose SOL has no intended monetary value. The application does not independently observe climate events, and the repository does not include proof that its Anchor program has been deployed to Devnet.

## Status at a glance

| Area                               | Current state                                                                                                                       |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Geographic interface               | Implemented with a local Natural Earth dataset, D3 canvas rendering, interaction, marker clustering, and accessible region controls |
| Market data                        | Nine fictional demo markets across six continents; no live climate feed                                                             |
| Wallet support                     | Phantom through a lightweight Solana Wallet Adapter context, fixed to Devnet                                                        |
| Purchase, claim, and refund client | Implemented, simulated before submission, and disabled until a program ID is configured                                             |
| Anchor program                     | Source and local-validator integration tests are included                                                                           |
| Default Devnet deployment          | **Not provided or verified**; `NEXT_PUBLIC_PROGRAM_ID` is blank by default                                                          |
| On-chain demo accounts             | Not automatically initialized or seeded; deployment alone is insufficient for trading                                               |
| Persistence                        | Seed files, browser `localStorage`, and an ephemeral in-memory API index; no database                                               |
| Production readiness               | Unaudited prototype with a centralized resolver and no oracle or dispute process                                                    |

## Product tour

- **Interactive climate atlas:** drag or use arrow keys to rotate, pinch/scroll or use `+`/`-` to zoom, select regions or markers, and reset the view. Idle rotation respects reduced-motion preferences.
- **Geographic discovery:** open markets are plotted by latitude and longitude. Nearby markers cluster, continent selection focuses the globe, and a text region selector provides a keyboard/mobile fallback.
- **Regional market drawer:** filter and search markets without navigating away from the globe. The drawer becomes a bottom sheet on small screens.
- **Market detail view:** inspect sample probability history, YES/NO probabilities, pool liquidity, volume, participants, countdown, resolution rules and source, evidence links, and recent fictional trades.
- **Devnet transaction flow:** connect a wallet, choose YES or NO, enter or quick-select a SOL amount, review the estimated payout, probability impact, and network fee, then simulate and submit the instruction.
- **Explicit transaction states:** preparing, simulation, wallet approval, confirmation, success with an Explorer link, and mapped failure messages. A submission lock prevents duplicate sends.
- **Settlement controls:** configured users can claim a winning pooled payout or refund both sides of a cancelled position.
- **Local portfolio:** confirmed browser-submitted positions are indexed per wallet in `localStorage`. This is a convenience view, not authoritative account state.

The primary interface remains useful in sample/read-only mode when no program is configured. Live transaction buttons explain what configuration is missing.

## Architecture

```text
Browser
  ├─ Next.js dashboard and responsive market drawer
  ├─ D3 canvas globe + local world-atlas topology
  ├─ Wallet Adapter (Phantom, Devnet)
  ├─ Sample market repository and pooled quote calculations
  └─ Solana instruction client
         │
         ├─ optional Devnet RPC ──> deployed climate_market program
         │                            ├─ PDA-owned state
         │                            └─ native-SOL market vaults
         │
         └─ Next.js API routes ──> seeded/in-memory off-chain metadata
```

### Source-of-truth boundaries

| Data                                                                                     | Authority today                                                                         |
| ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Seeded questions, coordinates, charts, evidence, trades, and UI probabilities            | `lib/markets/data.ts`; fictional sample metadata                                        |
| Browser-submitted position list                                                          | Wallet-scoped `localStorage`; convenience index only                                    |
| API-indexed activity                                                                     | Server process memory; cleared on restart/redeploy                                      |
| Deposited funds, position amounts, market status/outcome, and claims after configuration | The deployed Anchor program accounts                                                    |
| Real-world climate outcome                                                               | An authorized resolver applying the off-chain source and rules; no oracle is integrated |

The current UI does not reconcile its seeded pool figures against program accounts. Treat the program as authoritative whenever an on-chain deployment is in use.

## Repository layout

```text
app/
  api/                         Typed, Zod-validated metadata/activity routes
  globals.css                  Global Tailwind layers, focus, and motion styles
  layout.tsx                   Root metadata and styles
  page.tsx                     Dashboard entry point
components/
  globe/ClimateGlobe.tsx       Interactive D3 canvas globe
  layout/                      Navigation
  markets/                     Filters, lists, charts, drawer, and details
  portfolio/                   Browser-local positions
  providers/                   Market, position, and Solana contexts
  trading/                     Purchase review and settlement controls
  wallet/                      Wallet connect and balance components
hooks/
  useMarketProgram.ts          Transaction simulation/submission state machine
lib/
  geo/                         Continent centers and region helpers
  markets/                     Types, sample data, repository, and pooled math
  solana/                      Config, PDA derivation, IDL, encoding, and instructions
  validation/                  Runtime request and market schemas
programs/climate_market/
  src/                         Anchor accounts, instructions, events, and errors
tests/
  climate-market.ts            Local-validator Anchor integration suite
  *.test.ts(x)                 API, data/math, globe/drawer, and trading UI tests
```

## Prerequisites

### Web application

- Node.js 22.13 or newer
- npm (the lockfile is committed)
- A modern browser
- Phantom configured for Devnet if testing wallet behavior

### Anchor program development

The program toolchain is not installed by `npm ci` and is not vendored in this repository. Install:

- [Rust and Cargo through rustup](https://www.rust-lang.org/tools/install)
- [Solana/Agave CLI](https://solana.com/docs/intro/installation)
- [Anchor CLI through AVM](https://www.anchor-lang.com/docs/references/avm)

This codebase targets **Anchor 0.30.1** in `Anchor.toml`, `programs/climate_market/Cargo.toml`, and `package.json`. Use the matching CLI:

```bash
avm install 0.30.1
avm use 0.30.1
anchor --version
```

Anchor's [0.30.1 release notes](https://www.anchor-lang.com/docs/updates/release-notes/0-30-1) describe its matching Solana toolchain. Current Solana installation documentation may install a newer Agave release; compatibility with a newer CLI has not been verified here.

The environment used to assemble this prototype had Node.js and npm, but did not have `rustc`, `cargo`, `solana`, or `anchor`. Consequently, this README does not claim a successful program build, local-validator run, or Devnet deployment from that environment.

## Local setup

1. Install JavaScript dependencies:

   ```bash
   npm ci
   ```

2. Create the local environment file:

   ```bash
   cp .env.example .env.local
   ```

3. Leave `NEXT_PUBLIC_PROGRAM_ID` empty for sample/read-only mode, or configure a deployment as described below.

4. Start the application:

   ```bash
   npm run dev
   ```

5. Open [http://localhost:3000](http://localhost:3000).

For a production-style local check:

```bash
npm run typecheck
npm test
npm run build
npm run start
```

## Environment variables

| Variable                     | Required                  | Behavior                                                                                                                                       |
| ---------------------------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `NEXT_PUBLIC_SOLANA_NETWORK` | Recommended               | Must be `devnet`. Other values produce a configuration warning; this prototype does not support Mainnet.                                       |
| `NEXT_PUBLIC_SOLANA_RPC_URL` | No                        | Devnet HTTP(S) RPC endpoint. When blank or invalid, the client falls back to Solana's public Devnet endpoint. Public RPCs may be rate-limited. |
| `NEXT_PUBLIC_PROGRAM_ID`     | Required for transactions | Base58 address of the program actually deployed to Devnet. When blank or invalid, buy/claim/refund controls remain disabled.                   |
| `DATABASE_URL`               | No                        | Reserved for future PostgreSQL/Supabase work. It is not read by the current application.                                                       |

All variables prefixed with `NEXT_PUBLIC_` are exposed to the browser. Never put a private key, seed phrase, private RPC credential, or other secret in them. `.env.local` is ignored by Git; `.env.example` contains placeholders only.

## Fund a Devnet wallet

For a browser wallet, switch the wallet network to **Solana Devnet**, copy its public address, and use the [official Solana faucet](https://faucet.solana.com/). Never enter a wallet seed phrase into this application or a faucet.

With the Solana CLI installed:

```bash
solana config set --url devnet
solana address
solana airdrop 2
solana balance
```

Devnet airdrops are rate-limited and can fail temporarily. Devnet state may also be reset. Use only test wallets and test funds.

## Build and deploy the Anchor program

Deployment is an operator workflow, not part of `npm run build`. Review the program and use a dedicated Devnet authority; do not reuse a production wallet.

### 1. Verify the toolchain and wallet

```bash
rustc --version
cargo --version
solana --version
anchor --version
solana config set --url devnet
solana config get
solana balance
```

`Anchor.toml` defaults its provider to `Localnet`, so Devnet commands below override the cluster explicitly.

### 2. Build and synchronize the program ID

```bash
anchor build
anchor keys list
```

The same program address must be represented in all of these places:

1. the generated deployment keypair reported by `anchor keys list`;
2. `declare_id!(...)` in `programs/climate_market/src/lib.rs`;
3. `[programs.localnet]` and `[programs.devnet]` in `Anchor.toml`;
4. `NEXT_PUBLIC_PROGRAM_ID` in `.env.local` after deployment.

If you intentionally adopt the generated deployment keypair, synchronize the source/config and rebuild:

```bash
anchor keys sync
anchor build
anchor keys list
```

Inspect the resulting changes before deployment. The client IDL in `lib/solana/idl.ts` is maintained in source and takes its address from `NEXT_PUBLIC_PROGRAM_ID`; instruction/account changes must be kept synchronized with the generated Anchor IDL.

### 3. Deploy to Devnet and verify

```bash
anchor deploy --provider.cluster devnet
solana program show <PROGRAM_ID> --url devnet
```

The program address currently written in source and `Anchor.toml` is configuration, not evidence of a successful deployment. Only the Devnet query above (or an Explorer account lookup) verifies deployment.

### 4. Initialize and seed program accounts

A freshly deployed program has no protocol or markets. Before the browser can trade, an operator must:

1. call `initialize_protocol` once with an authority signer and a non-default resolver;
2. call `create_market` for each numeric market ID, question hash, close time, and resolution time;
3. call `fund_market` as the market authority so the desired YES/NO pools exist;
4. keep the off-chain market metadata and `onchainMarketId` values aligned with those accounts.

The seeded UI uses numeric IDs `1001`–`1009`, but the repository does **not** include a Devnet bootstrap/admin script and does not prove those accounts exist. `tests/climate-market.ts` demonstrates the Anchor client calls against Localnet and can be used as a reference when writing an operator script. The public UI currently constructs only buy, claim, and refund instructions—not initialize, create, fund, close, or resolve instructions.

### 5. Point the web client at the verified deployment

```env
NEXT_PUBLIC_SOLANA_NETWORK=devnet
NEXT_PUBLIC_SOLANA_RPC_URL=https://your-devnet-rpc.example
NEXT_PUBLIC_PROGRAM_ID=<PROGRAM_ID>
```

Restart the Next.js server after changing public environment variables. A configured program ID enables the transaction client, but transactions will still fail simulation if the corresponding protocol, market, vault, or position accounts are absent or incompatible.

## Protocol design

The Anchor program accepts native SOL in lamports and implements a pooled binary market. It is not an order book, AMM, tokenized share system, or secondary market.

### Program-derived accounts

| Account          | PDA seeds                                       | Purpose                                                                         |
| ---------------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| `ProtocolConfig` | `["protocol"]`                                  | Protocol authority, authorized resolver, market count, bump                     |
| `Market`         | `["market", market_id.to_le_bytes()]`           | Deadlines, status/outcome, pool totals, paid total, resolver, and question hash |
| `MarketVault`    | `["vault", market_pubkey]`                      | Program-owned account holding native SOL while preserving its rent reserve      |
| YES `Position`   | `["yes_position", market_pubkey, owner_pubkey]` | User's cumulative YES deposit                                                   |
| NO `Position`    | `["no_position", market_pubkey, owner_pubkey]`  | User's cumulative NO deposit                                                    |
| `ClaimRecord`    | `["claim", market_pubkey, owner_pubkey]`        | One settlement record per user/market, preventing a second claim or refund      |

### Instructions and authorization

| Instruction                          | Who may call           | Main conditions/effect                                                                                            |
| ------------------------------------ | ---------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `initialize_protocol(resolver)`      | Initial authority      | Creates the singleton protocol PDA and stores authority/resolver                                                  |
| `create_market(...)`                 | Protocol authority     | Requires a nonzero question hash, future close, and resolution time at/after close; creates market and vault PDAs |
| `fund_market(yes, no)`               | Market authority       | Before close; deposits at least one positive side and records authority positions                                 |
| `buy_yes(amount)` / `buy_no(amount)` | Any funded signer      | Positive native-SOL deposit while the market is open and before its deadline                                      |
| `close_market()`                     | Any signer             | Changes Open → Closed only after the close timestamp                                                              |
| `resolve_market(decision)`           | Stored resolver        | Only after Closed and the resolution timestamp; YES/NO needs nonzero winning liquidity, otherwise cancel          |
| `claim_winnings()`                   | Winning position owner | Withdraws the proportional payout once after YES/NO resolution                                                    |
| `refund_cancelled()`                 | Position owner         | Returns the user's YES + NO deposits once after cancellation                                                      |

The resolver is copied from `ProtocolConfig` into each market at creation. There is currently no resolver-rotation or market-update instruction.

### Probability and payout

Before resolution, the displayed pooled probabilities are:

```text
YES probability = YES pool / (YES pool + NO pool)
NO probability  = NO pool  / (YES pool + NO pool)
```

The frontend displays an empty pool as 50/50. Probabilities are informational ratios, not oracle confidence or guaranteed execution prices. A new deposit changes the displayed ratio, and later deposits can change the eventual payout.

After a YES or NO resolution:

```text
user payout = user winning position × total market pool ÷ winning-side pool
```

The program stores deposits as `u64` lamports, performs the multiplication in checked `u128` arithmetic, converts the result back to `u64`, and uses integer division. Division therefore rounds **down to the nearest lamport**. Any sub-lamport remainder remains as vault dust. The program also enforces `total_paid_amount <= total_pool_amount`.

For a cancelled market:

```text
user refund = user YES position + user NO position
```

No platform fee is currently deducted; users still pay normal Solana network fees.

## Resolution flow

The contract cannot determine a hurricane, drought, temperature, or crop outcome by itself.

1. The application stores human-readable rules, source URLs, and evidence off-chain. The market account stores a question hash and timestamps, not the full evidence package.
2. After the close timestamp, any signer may call `close_market` to stop trading on-chain.
3. At or after the resolution timestamp, the configured resolver evaluates the published source/rules and submits `Yes`, `No`, or `Cancelled`.
4. The program records the result and emits a resolution event. YES/NO is rejected if the selected winning pool is empty.
5. Winning users claim proportionally; cancelled-market users refund their original deposits. The claim PDA prevents repeat settlement.
6. An off-chain indexer should retain the resolver transaction, source snapshot, and user-facing audit trail.

This repository has no resolver dashboard, automated climate ingestion, dispute window, oracle adapter, or multisig policy. Resolution currently requires custom operator tooling.

## API routes

All routes validate input with Zod and return Devnet/sample metadata. They do not read program accounts.

| Method | Route                            | Purpose                                                                                               |
| ------ | -------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `GET`  | `/api/markets`                   | List markets; supports `search`, `category`, `continent`, `status`, `featured`, `limit`, and `offset` |
| `GET`  | `/api/markets/:id`               | Get one market by ID or slug                                                                          |
| `GET`  | `/api/markets/region/:continent` | List a continent's markets; continent names or hyphenated slugs are accepted                          |
| `GET`  | `/api/markets/:id/history`       | Return fictional probability/liquidity history                                                        |
| `GET`  | `/api/users/:wallet/positions`   | Return seeded sample positions for a validated 32-byte Solana public key                              |
| `GET`  | `/api/users/:wallet/activity`    | Return seeded plus process-memory activity for a wallet                                               |
| `POST` | `/api/index-transaction`         | Idempotently record validated transaction metadata in process memory                                  |

Success responses use:

```json
{
  "data": {},
  "meta": {
    "generatedAt": "2026-01-01T00:00:00.000Z",
    "network": "devnet",
    "isDemo": true,
    "dataLabel": "SAMPLE DATA",
    "disclaimer": "..."
  }
}
```

Errors use the same `meta` object and an `error` object containing `code`, `message`, and optional validation `details`.

`POST /api/index-transaction` accepts a wallet, market ID, numeric on-chain market ID, activity type/status, optional side and decimal lamport string, a 64-byte base58 transaction signature, and optional timestamp/failure reason. It does not verify the signature against RPC and explicitly returns `authoritative: false`. Its in-memory records disappear when the server process restarts or a serverless instance is replaced.

## Commands and tests

| Command                | Purpose                                                                                   |
| ---------------------- | ----------------------------------------------------------------------------------------- |
| `npm run dev`          | Start Next.js development mode                                                            |
| `npm run build`        | Create a production Next.js build                                                         |
| `npm run start`        | Serve the production build                                                                |
| `npm run lint`         | Run ESLint checks                                                                         |
| `npm run typecheck`    | Run strict TypeScript checking without emitting files                                     |
| `npm test`             | Run the root Vitest suites once                                                           |
| `npm run test:watch`   | Run Vitest in watch mode                                                                  |
| `npm run test:anchor`  | Run `anchor test` and the Localnet integration suite; requires the full program toolchain |
| `npm run format:check` | Check Prettier formatting                                                                 |
| `npm run format`       | Rewrite supported files with Prettier                                                     |
| `npm run check`        | Format check, lint, typecheck, root tests, and web build                                  |

The root tests cover pooled arithmetic and validation, sample data/repository behavior, API routes, globe-to-drawer interaction, market selection, wallet/amount validation, pending/success/failure transaction states, duplicate-confirm protection, and closed-market behavior.

The Anchor Localnet suite covers protocol initialization, market creation/funding, YES and NO purchases, zero-value and incorrect-PDA rejection, deadline enforcement, unauthorized/early resolution, YES/NO/cancelled outcomes, losing and double-claim rejection, proportional payouts, and cancellation refunds. `anchor test` builds and deploys to a temporary local validator; it does not prove Devnet deployment.

## Security properties

Implemented safeguards include:

- explicit signer, authority, resolver, PDA seed/bump, market, vault, position, and claim-record validation;
- checked `u64` pool accounting and `u128` payout multiplication;
- positive-deposit, wallet-balance, status, deadline, and resolution-time checks;
- winning-liquidity requirements and a one-claim-per-user/market record;
- vault withdrawals limited to settlement/refund paths while retaining rent reserve;
- client-side simulation before wallet submission, confirmation tracking, and duplicate-submission locking;
- no private keys, seed phrases, or secrets in frontend code;
- Devnet labels, Explorer links, sample-data labels, and an experimental-service disclaimer.

These controls are not a substitute for an independent audit. The program has no pause authority, upgrade-governance policy, resolver rotation, dispute mechanism, or formal verification. Operators must secure deployment and resolver keys separately.

## Accessibility

- The globe is focusable and supports arrow-key rotation, keyboard zoom, reset, and a text region selector.
- Pointer events support mouse, touch drag, and pinch zoom.
- Idle animation and CSS transitions respect `prefers-reduced-motion`.
- Controls use visible focus indicators, labels, `aria-pressed`, dialog semantics, live status/error regions, and descriptive transaction states.
- YES and NO are always identified with text, not color alone.
- The regional drawer becomes a large bottom sheet on mobile.

The canvas is supplemented by textual navigation and market lists, but it is not a semantic map of every rendered marker. Continue testing with keyboard-only navigation, screen readers, zoom, and high-contrast settings before any public release.

## Known MVP limitations

- No default or verified Devnet deployment is supplied, and `NEXT_PUBLIC_PROGRAM_ID` is intentionally blank.
- Deploying the program does not initialize the protocol or create/fund the UI's sample market PDAs. No admin/bootstrap script or operator interface is included.
- The web client does not fetch or reconcile on-chain market accounts; displayed pools, probabilities, statuses, positions, charts, evidence, and recent trades are sample/off-chain data.
- Browser positions are stored in wallet-scoped `localStorage`; API activity is held in process memory. Neither is a durable index or source of truth.
- `DATABASE_URL` is unused. There is no PostgreSQL/Supabase schema, event worker, retry queue, or chain reorganization handling.
- The climate data is not live. Source links are proposed resolution references, not proof that the sample outcomes or probabilities are real.
- Resolution is centralized in one signer, with no oracle, multisig, dispute period, or governance process.
- Only native SOL is supported. There is no USDC vault, token-account abstraction, treasury fee, order book, secondary trading, or early exit.
- The frontend IDL/instruction definitions are source-maintained and must be synchronized when the Anchor interface changes.
- Integer division can leave vault dust, and the program has no dust sweep or account-closing flow.
- Devnet is rate-limited and resettable; it is not a reliability or security analogue for Mainnet.
- Rust, Solana, and Anchor tooling must be installed separately. Program tests and deployment cannot run in a Node-only environment.
- The program and end-to-end operator workflow have not been independently audited.

## Roadmap

- Replace sample metadata with PostgreSQL or Supabase repositories and durable event/activity indexing.
- Reconcile UI state from program accounts and indexed events rather than browser storage.
- Add signed climate-source adapters and oracle integrations, source snapshots, resolver multisig, disputes, and audit trails.
- Add an operator console and repeatable scripts for initialization, market creation/funding, closure, and resolution.
- Generalize settlement assets to support token-program-backed USDC without changing market-domain APIs.
- Generate and validate the frontend IDL/client from each Anchor build in CI.
- Add program fuzz/property tests, deployment smoke tests, monitoring, upgrade controls, and an external security audit.
- Improve marker-cluster exploration and expand semantic map alternatives for assistive technology.

## Disclaimer

Klashi is experimental software for development and demonstration. It is not investment advice, insurance, a regulated market, or a production hedging service. Do not use real funds, do not rely on fictional climate data, and never share a wallet seed phrase.
