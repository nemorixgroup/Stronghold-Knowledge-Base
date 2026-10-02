# Balance Queries

**Phase:** 2 - SHx Asset Operations & Onboarding
**Status:** Done (`0.0.5-dev`)
**Implementation:** [`lib/src/wallet/shx_balance.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/wallet/shx_balance.dart), [`test/src/wallet/shx_balance_integration_test.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/test/src/wallet/shx_balance_integration_test.dart)

## What This Is

`ShxBalance` reads an account's XLM and SHx balances from Horizon. This document covers three decisions: why it is its own class, why a missing trustline returns a value instead of an error, and why matching on asset code alone would be a mistake.

## Decision 1: A Separate Class, Not Part of ShxWallet

Same reasoning already applied to `ShxTrustline`/`ShxPayment` in Phase 1, and to `establishShxTrustline` in the previous `docs-sdk` entry: reading an account's current state and changing its lifecycle are different responsibilities. `ShxWallet` creates, funds, and upgrades accounts; `ShxBalance` only reads. Neither method in this class signs or submits a transaction.

## Decision 2: No Trustline Is Not an Error

`getShxBalance()` returns `'0'` when the account has no SHx trustline, rather than throwing. This matches how the rest of the SDK already models the account lifecycle: `ShxAccountStatus.funded` is an expected, normal state an account can sit in indefinitely before becoming `shxReady` (see [`ShxAccount`](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-1/composition-not-fork/README.md)). A balance query is something a UI will call routinely while an account is in that state, for example, to decide whether to show an "activate SHx" prompt, so treating the absence of a trustline as an exception would force every caller to wrap a routine, expected case in a try/catch.

Contrast this with `getXlmBalance()`, which does throw `StrongholdException` if the account cannot be loaded at all: a nonexistent account is a genuine error condition, not an expected intermediate state the way a missing SHx trustline is.

## Decision 3: Match on Asset Code and Issuer Together

`getShxBalance()` filters Horizon's balance list on both `assetCode == 'SHX'` and `assetIssuer == StrongholdConstants.shxIssuerAccountId`, never on the code alone. This is the same principle already established in [Module 02 of the Knowledge Base](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/module-02-shx-token/README.md) and reinforced in the [SHx Trustline](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/shx-trustline/README.md) decision: anyone can issue an asset with the code `SHX` from a different account, and it would be a lookalike, not SHx. A balance method that matched on code alone could report a nonzero "SHx" balance for an account that only holds an unrelated asset sharing that code, silently wrong in a way that would be easy to miss in testing.

## Known Limitation (Inherited)

Verifying a genuinely nonzero SHx balance requires the real SHx issuer, which only exists on Mainnet (see [SHx Trustline](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/shx-trustline/README.md) for the full `op_no_issuer` finding). The zero-balance path (no trustline) is fully testable on Testnet and covered by a real integration test; the nonzero path is marked `skip`, pending a funded Mainnet test account, same as `establishShxTrustline`.

## Related

- [SHx Trustline](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/shx-trustline/README.md)
- [Account Bootstrap Helpers](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/account-bootstrap-helpers/README.md)
