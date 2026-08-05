# SAC vs. Governance Contract

**Phase:** 1 - Architecture & Core Setup  
**Status:** ✅ Done (`0.0.1-dev`)  
**Implementation:** [`lib/src/core/stronghold_network.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/core/stronghold_network.dart), [`lib/src/governance/shx_vote.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/governance/shx_vote.dart)

## What This Is

Stronghold's own governance documentation (`docs.shx.stronghold.co/shx/shx-governance-rules`) references a Soroban Contract ID (`CCKCKCPHYVXQD4NECBFJTFSCU2AMSJGCNG4O6K4JVRE2BLPR7WNDBQIQ`) in the context of SHx governance. Before writing any governance code, this SDK's architecture depended on knowing what that contract actually does.

## The Investigation

Rather than assume the documentation was complete, the contract was inspected directly on Mainnet:

```
stellar contract inspect --id CCKCKCPHYVXQD4NECBFJTFSCU2AMSJGCNG4O6K4JVRE2BLPR7WNDBQIQ --network public
```

The result did not match the expectation. The contract's spec contains only the standard SEP-41 token interface (`balance`, `transfer`, `transfer_from`, `approve`, `allowance`, `mint`, `burn`, `burn_from`, `clawback`, `set_admin`, `set_authorized`, `decimals`, `name`, `symbol`), with no lock, delegate, or vote functions of any kind. Further confirmation came from the contract's own metadata:

- **Source code:** `stellar/rs-soroban-env`, Stellar protocol code, not anything Stronghold authored
- **Creation date:** February 21, 2024, one day after Soroban activated on Mainnet (February 20, 2024)

This is the **Stellar Asset Contract (SAC)**, the wrapper Soroban automatically generates for every classic Stellar asset so it can be used from smart contracts. It is protocol-level infrastructure, not a Stronghold governance contract.

## Why This Mattered for the SDK's Design

Had this not been checked, the natural (and wrong) assumption would have been to build governance voting as a Soroban contract client, calling `lock`/`delegate`/`vote`-style functions on this Contract ID. None of those functions exist there.

## Decision

Governance voting is implemented via classic Stellar `ManageData` operations instead, matching what Stronghold's own [voting tool](https://vote.stronghold.co) actually uses:

```
name  => "SHX::PROP:<proposal number>"
value => vote position, e.g. "FOR" or "FOR,40"
```

No Soroban interaction is required for voting at all. The SAC contract identified above is not used anywhere in the current governance implementation; it remains relevant only as the entry point for SHx if a future Soroban contract ever needs to receive or transfer SHx directly.

## Related

- [Composition, Not a Fork](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/tree/main/docs-sdk/phase-1/composition-not-fork/README.md)
- Stronghold/SHx [Knowledge Base](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/README.md), Module 02 (The SHx Token), section on the Soroban wrapper
