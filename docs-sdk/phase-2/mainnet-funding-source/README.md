# Mainnet Funding Source

**Phase:** 2 - SHx Asset Operations & Onboarding  
**Status:** ✅ Done (`0.0.3-dev`)  
**Implementation:** [`lib/src/wallet/shx_wallet.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/main/lib/src/wallet/shx_wallet.dart), `ShxWallet.createAndFund`

## What This Is

Unlike Testnet, Mainnet has no faucet: creating an account requires real XLM from a real source. This document records why that source is a parameter the caller supplies, rather than something the SDK manages or assumes.

## The Question

`createAndFund()` needs a funded account to pay for the new account's starting balance. The open question was whether the SDK should own any part of that funding logic (a built-in treasury concept, a default funding strategy) or stay out of it entirely.

## Options Considered

1. **SDK-managed funding (e.g. a configured treasury account, or a default funding strategy).** Rejected. Every app built on SHx will have a different answer for where XLM comes from: a company treasury, a user's own prior deposit, an exchange withdrawal, a purchase flow. Baking any one of these in as a default would be wrong for most callers, and configuring it away would add API surface for no real benefit over just asking for a `KeyPair` directly.
2. **Caller-supplied `fundingSourceKeyPair` parameter.** Chosen. `createAndFund()` only builds and submits the `CreateAccountOperation`; it never decides where the funding comes from. This keeps the SDK generic across use cases (see [Composition, Not a Fork](../../phase-1/composition-not-fork/README.md) for the same generality principle applied at the architecture level), payments, remittances, tokenization, or anything else Stronghold enables on SHx, rather than assuming any one of them.

## Decision

`fundingSourceKeyPair` is a required parameter on `createAndFund()`. The developer's own financing plan and sequencing (when to fund, from where, in what order relative to other app logic) is entirely their decision; the SDK's job ends at building a correct, signed transaction from what it is given.

## Related

- [Account Bootstrap Helpers](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/account-bootstrap-helpers/README.md)
