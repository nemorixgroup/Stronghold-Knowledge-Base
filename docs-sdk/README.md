# SDK Technical Decisions

This section documents the implementation decisions, technical rationale, and key references behind **stronghold_flutter_sdk**, the first native Flutter/Dart SDK for the SHx token ecosystem on Stellar.

Every decision documented here is grounded in official Stronghold/SHx sources, official Stellar network data, or direct on-chain verification (via `stellar contract inspect` against Mainnet), not assumed from documentation or marketing copy. Where a finding contradicts or refines existing documentation, that is stated plainly.

## Why This Documentation Exists

Building a production-quality SDK for SHx requires making decisions that are not always obvious, which Stellar package to build on, whether a documented Contract ID actually does what it claims, why a given architecture was chosen over a simpler-looking alternative.

This documentation answers those questions for:

- **Contributors** who want to understand the rationale before submitting a pull request
- **Developers** who want to verify that the SDK follows verified, real SHx/Stellar behavior rather than assumptions
- **Auditors** who need a clear trail from specification (or on-chain fact) to implementation

## Structure

```
docs-sdk/
  README.md               <- You are here
  phase-1/                <- Architecture & Core Setup
  phase-2/                <- SHx Asset Operations & Onboarding
  phase-3/                <- Governance (Voting)
  phase-4/                <- Escrow Contract Client
  phase-5/                <- Bridge Status & Tracking
  phase-6/                <- Testing, Documentation & pub.dev v1.0
```

## Phase 1 - Architecture & Core Setup

| Feature | Description | Status |
|---|---|---|
| [Composition, Not a Fork](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-1/composition-not-fork/README.md) | Why the SDK depends on `stellar_flutter_sdk` as a package instead of forking it | ✅ Done |
| [SAC vs. Governance Contract](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-1/sac-vs-governance-contract/README.md) | The on-chain finding that Stronghold's documented "governance Contract ID" is actually the standard Stellar Asset Contract, and what that means for how voting is implemented | ✅ Done |
| [Linter Selection](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-1/linter-selection/README.md) | Why `very_good_analysis` replaced `flutter_lints` | ✅ Done |

## Phase 2 - SHx Asset Operations & Onboarding

| Feature | Description | Status |
|---|---|---|
| Account Bootstrap Helpers | Granular, composable helpers vs. one monolithic onboarding function | 🔄 Next |
| Mainnet Funding Source | Why the funding `KeyPair` is a caller-supplied parameter, not SDK-managed | ⏳ Pending |
| Trustline & Payment | Already implemented in the Phase 1 scaffold | ⏳ Pending write-up |
| Path Payment | Cross-asset SHx conversion | ⏳ Pending |

## Phase 3 - Governance (Voting)

| Feature | Description | Status |
|---|---|---|
| ManageData Voting Format | Already implemented in the Phase 1 scaffold | ⏳ Pending write-up |
| Vote Reading | Decoding an account's current vote from Horizon | ⏳ Pending |

## Phase 4 - Escrow Contract Client

| Feature | Description | Status |
|---|---|---|
| Escrow Client (lock/unlock/extendTtl/getEscrow) | Already scaffolded against the verified on-chain spec | ⏳ Pending write-up |
| Storage Durability | PERSISTENT vs. TEMPORARY, open question pending Testnet verification | ⏳ Pending |
| Error Extraction | Exact on-chain error shape, open question pending a real failing call | ⏳ Pending |

## Phase 5 - Bridge Status & Tracking

| Feature | Description | Status |
|---|---|---|
| Axelarscan as Source of Truth | Why bridge status reads from the Axelarscan GMP API instead of a direct contract read | ⏳ Pending |

## Phase 6 - Testing, Documentation & pub.dev v1.0

| Feature | Description | Status |
|---|---|---|
| Release Process | pana/dry-run verification steps before each pub.dev publish | ⏳ Pending |

## Related

- [stronghold_flutter_sdk](https://github.com/nemorixgroup/stronghold-flutter-sdk) - the SDK itself
- [Stronghold/SHx Knowledge Base](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/README.md) - official source documentation
