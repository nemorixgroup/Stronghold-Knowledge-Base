# Composition, Not a Fork

**Phase:** 1 - Architecture & Core Setup  
**Status:** ✅ Done (`0.0.1-dev`)  
**Implementation:** [`pubspec.yaml`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/pubspec.yaml)

## What This Is

`stronghold_flutter_sdk` depends on [`stellar_flutter_sdk`](https://github.com/Soneso/stellar_flutter_sdk) (Soneso, MIT license) as a normal pub.dev package. This document records why that was chosen over forking `stellar_flutter_sdk` and building directly on top of the fork, which was the first option considered.

## The Question

SHx has no blockchain of its own; it is an asset issued on Stellar. Every operation this SDK needs (trustlines, payments, Soroban contract calls) is something `stellar_flutter_sdk` already implements and actively maintains. The open question was whether to fork that package (giving full control over its internals) or depend on it as-is and build only the SHx-specific layer on top.

## Options Considered

1. **Fork `stellar_flutter_sdk`.** Rejected. `stellar_flutter_sdk` is funded by the Stellar Public Goods Program and actively maintained by Soneso, tracking Stellar protocol changes, XDR updates, and Soroban RPC changes as they happen. Forking would mean re-doing that maintenance ourselves, forever, for a general-purpose Stellar client that has nothing to do with SHx specifically. It would also work against the wider Stellar developer ecosystem rather than with it, fragmenting tooling instead of strengthening a shared dependency.
2. **Depend on it as a package, add an SHx-specific layer on top.** Chosen. `stronghold_flutter_sdk` adds only what is genuinely SHx-specific: asset constants, trustline/payment helpers scoped to SHx, the governance voting format, and the escrow contract bindings. If a gap is ever found in the base SDK during development, the correct response is a pull request upstream to Soneso for that specific piece, not a fork of the whole package.

## Decision

Composition over forking. `stronghold_flutter_sdk`'s `pubspec.yaml` declares `stellar_flutter_sdk` as a standard dependency (currently `^3.3.0`). This is also the same pattern already used across the rest of Nemorix Group's SDK portfolio: none of `hedera_flutter_sdk`, `xrpl_flutter_sdk`, or `avalanche_flutter_sdk` fork a lower-level chain client either.

## Related

- [SAC vs. Governance Contract](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-1/sac-vs-governance-contract/README.md), a finding that reinforced this decision: even Stronghold's own governance Contract ID turned out to be protocol-level Stellar code, not something Stronghold wrote, further evidence that the SHx-specific surface area is genuinely small
