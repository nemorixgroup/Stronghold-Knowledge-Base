# SHx Payment

**Phase:** 2 - SHx Asset Operations & Onboarding  
**Status:**: ✅ Done `0.0.7-dev` (see the [CHANGELOG](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/CHANGELOG.md))  
**Files:** [`lib/src/asset/shx_asset.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/asset/shx_asset.dart), [`lib/src/core/stronghold_exception.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/lib/src/core/stronghold_exception.dart), [`test/src/asset/shx_payment_test.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/test/src/asset/shx_payment_test.dart), [`test/src/core/shx_payment_exception_test.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/test/src/core/shx_payment_exception_test.dart), [`test/src/asset/shx_payment_memo_test.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/test/src/asset/shx_payment_memo_test.dart), [`test/src/asset/shx_payment_integration_test.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/test/src/asset/shx_payment_integration_test.dart), [`example/phase2/shx_payment_example.dart`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/example/phase2/shx_payment_example.dart)

## Summary

`ShxPayment.send()` sends SHx from a `shxReady` account to another account, with an optional text memo, and returns the transaction hash. Before spending a fee it checks that the destination exists and holds the SHx trustline, and it reports every failure as a typed `ShxPaymentException`. It is the payment step of the onboarding chain built in Phase 2 ([trustline](../shx-trustline/README.md), [balances](../balance-queries/README.md), [bootstrap](../bootstrap-for-shx/README.md)).

This page records seven decisions and what the tests do and do not prove. A successful payment has not been verified end to end: it needs Mainnet.

## Design Decision: Stronghold Has No Payment API for SHx

Checked on 2026-11-13, the official Stronghold sources say nothing about how to send SHx:

- The SHx documentation (`docs.shx.stronghold.co`) lists 18 pages in its `llms.txt`: governance rules, the 7 proposals, the voting process, the Ecosystem Development Program, and community guidelines. None is about sending, receiving, or holding SHx.
- The `strongholdpay` GitHub organization has 10 repositories. None contains SHx payment code. `react-native-sdk` is for Stronghold Pay and has not changed since December 2022. `kelp`, `go`, `edge-react-gui`, `wallet-react-native`, `p2pb2b`, `xlm-aggregator`, and `compliance-protocol` appear to be copies of third-party Stellar projects from 2018 and 2019.
- The SHx page on `stronghold.co` says SHx is built on Stellar and held in Stellar wallets (Freighter, Lobstr, Ledger, StellarTerm). The code snippets in its "Built for developers" section are for Stronghold Pay operations (payment source, charge, tip), not for on-chain SHx.

So a SHx payment is an ordinary Stellar payment of a Stellar asset. The SHx-specific data (asset code, issuer) comes from Stronghold, and the mechanics come from the Stellar documentation and from `stellar_flutter_sdk`: the same split as in every earlier release. Stellar's documentation says that a payment of a Stellar asset between two accounts uses the payment operation.

## Design Decision: The Method Lives in `ShxPayment` and Returns the Hash

`send()` is in `ShxPayment`, next to `buildPaymentOperation`, and not in `ShxWallet`: `ShxWallet` changes the account lifecycle, `ShxBalance` reads, and `ShxPayment` builds and sends payments. The same separation by responsibility as in the earlier releases.

It returns the transaction hash, because a payment does not change the account's status (the source stays `shxReady`), and a wallet screen needs a receipt. The hash is computed locally before submitting, from the signed transaction, and it is the one returned on success. A Testnet test checks that this recipe (build, sign, hash) gives the same hash as the one Horizon reports.

## Design Decision: `send()` Never Retries, and Only One Error Means "Maybe"

Stellar's documentation says that a transaction that fails with `transaction_failed` will, in almost every case, never succeed; the one exception is `tx_bad_seq`, which can become valid later. Its error-handling guide goes further: after a `tx_bad_seq`, resubmitting blindly with a refreshed sequence number can make more than one payment when only one was intended. So `send()` makes one attempt, and any retry is the caller's decision.

What a failure means for the money depends on where it happens:

1. **Raised before submitting** (invalid input, the destination lookup, a failed account load): nothing was sent.
2. **The network answers with result codes:** a `tx_failed` means one operation failed and none was applied, according to the Stellar documentation of transaction result codes. The fee may still be charged: Stellar's guide on operations says the fee is taken from the source account regardless of success or failure (read in a mirror of the documentation, not on the live page).
3. **No definitive answer:** `submitTransaction` throws, or the network reports a failure without result codes. The payment may or may not have been applied. This is `outcomeUnknown`, and it carries the transaction hash so the caller can look it up before deciding to send again.

Any exception from `submitTransaction` is counted as unknown, which is the conservative choice: some of them may be definitive failures. What the SDK throws on a timeout was not verified, because a timeout was not simulated.

## Design Decision: The Destination Is Checked Before Sending

The most likely failures are a destination that does not exist and a destination without the SHx trustline. Stellar documents both (`op_no_destination`, `op_no_trust`) and says to make sure the receiver holds the trustline before sending a non-XLM asset. A failed transaction can still cost a fee, so `send()` looks up the destination first and reports these two cases without sending.

The trustline is matched on asset code and issuer together, never on the code alone, the same rule as in [Balance Queries](../balance-queries/README.md): anyone can issue an asset with the code `SHX`.

The cost is one extra Horizon call. The check is not atomic with the submission: the trustline could change in between, so the network remains the final authority and the same two codes can still come back from it. The destination's trustline limit is not checked; `op_line_full` comes from the network.

A consequence for testing: nobody can hold SHx on Testnet, so every valid `send()` call there stops at `destinationNotFound` or `destinationNoTrustline`, before anything is submitted.

## Design Decision: The Error Codes Mirror Stellar's, Without Authorization

`ShxPaymentException.fromResultCodes` maps Horizon result codes to `ShxPaymentErrorCode`. It is public so that callers who build their own transaction with `buildPaymentOperation` get the same typed errors, and so it can be tested without a network.

| Stellar result code | SDK code |
|---|---|
| `tx_insufficient_balance` | `insufficientXlmForFee` |
| `op_no_destination` | `destinationNotFound` |
| `op_no_trust` | `destinationNoTrustline` |
| `op_underfunded` | `insufficientShx` |
| `op_line_full` | `destinationLimitReached` |
| anything else (`op_malformed`, `op_no_issuer`, `op_src_no_trust`, `tx_bad_seq`, `tx_bad_auth`, `tx_insufficient_fee`, the authorization codes) | `rejected`, with the raw codes |

Three points about this table:

- **No authorization codes.** `op_not_authorized` and `op_src_not_authorized` are in Stellar's list for the Payment operation, but SHx's issuer cannot cause them. A Horizon query for the asset, run on 2026-11-13, returned `auth_required: false`, `auth_revocable: false`, `auth_immutable: true`, and `auth_clawback_enabled: false`, with 94,555 authorized trustlines and none unauthorized. Stellar's account documentation says that when `auth_immutable` is set, none of the flags can be changed, so this cannot change. If one of those codes ever appeared, it would fall under `rejected`.

  ```bash
  curl -s "https://horizon.stellar.org/assets?asset_code=SHX&asset_issuer=GDSTRSHXHGJ7ZIVRBXEYE5Q74XUVCUSEKEBR7UCHEUUEK72N7I7KJ6JH"
  ```

- **`insufficientXlmForFee` is its own code.** Stellar describes `tx_insufficient_balance` as "fee would bring account below reserve". So an account holding exactly its minimum XLM cannot pay even the fee: the reason [`bootstrapForShx()`](../bootstrap-for-shx/README.md) asks the caller to add headroom above the minimum.
- **`op_underfunded` is described differently on two Stellar pages.** The page of result codes talks about lumens; the page of operations talks about funds of the asset being sent, minus selling liabilities. The second is the one used: it means not enough SHx.

`tx_bad_seq` falls under `rejected` on purpose: two payments sent at once from the same account can collide, and the SDK does not retry.

## Design Decision: Input Rules

- **Amount.** A plain decimal with up to 7 decimals, above zero, at most 922337203685.4775807. Stellar stores amounts as 64-bit integers scaled by ten million, which gives that limit. The amount is parsed exactly into stroops with `BigInt`, not with `double`: a `double` cannot hold 19 significant digits, and Stellar's documentation recommends big-number arithmetic for the same reason. The forms `.5`, `1.`, `1e3`, signs, spaces, and non-ASCII digits are rejected.
- **Destination.** Only classic `G` accounts. The shape is checked with a pattern, and the checksum by decoding the address with `KeyPair.fromAccountId` (a corrupted checksum is rejected, as a test shows). Contract (`C`) addresses are rejected because Stellar's documentation says its payment operations cannot have contract addresses as source or destination: paying a contract goes through the asset's Stellar Asset Contract. Muxed (`M`) addresses are valid for a payment, but the destination lookup would need to extract the underlying account, so they are left out of this release.
- **Memo.** Text only, 1 to 28 bytes of UTF-8, counted in bytes and not in characters (28 emoji are 112 bytes). The SDK documents a text memo as up to 28 bytes of UTF-8, and a test with no network shows it agrees with this check: it writes the exact UTF-8 bytes into the transaction and rejects the same memos. An empty memo is rejected; pass `null` for none. Which memo exchanges require to credit SHx deposits was not verified, and a memo of another type (id, hash) is not supported.

## Design Decision: Testnet Proves What It Can by Stopping Early

The real SHx issuer only exists on Mainnet (see [SHx Trustline](../shx-trustline/README.md)), so a payment can never reach the submission stage on Testnet. The integration tests use that instead of fighting it:

- **Valid inputs get past validation.** Validation runs before the destination lookup, so a valid call to an account that was never funded ends in `destinationNotFound`, not in `invalidAmount` or `invalidMemo`. That is how the amount limits (one stroop and the maximum) and the 28-byte memos are shown to be accepted.
- **Nothing is sent when the destination cannot receive SHx.** The source account's sequence number and XLM balance are the same before and after.
- **The hash recipe is checked with an XLM payment** that Testnet can complete. It exercises the SDK calls that `send()` makes, not `send()` itself.

## What 0.0.7-dev Tests Actually Check

Unit tests, no network (66). Every `send()` case passes a real `StellarSDK` instance but must be rejected before any request:

1. `maxMemoBytes` is 28.
2. `buildPaymentOperation` pays SHx (asset code and issuer), to the given destination, for the given amount.
3. A source that is `pending` or `funded` is rejected as `sourceNotReady`, and that check runs before the others (3 tests).
4. Nine malformed destinations are rejected (empty, not an address, one character short, one character long, lowercase, a leading space, an `M` prefix, a contract address, a corrupted checksum), and paying the source itself is rejected (10 tests).
5. Twenty-two invalid amounts are rejected, including eight decimals, one stroop above the maximum, and Arabic-Indic digits.
6. Four invalid memos are rejected: empty, 29 ASCII bytes, 8 emoji (32 bytes), and 15 two-byte characters (30 bytes).
7. `ShxPaymentException` is a `StrongholdException`, keeps its fields, and names its code in `toString` (3 tests).
8. `fromResultCodes` classifies 14 result-code cases, and defaults the operation codes to an empty list (15 tests).
9. The SDK's text memo writes the exact UTF-8 bytes of four valid memos into a signed transaction, and rejects three that are too long (7 tests).

Integration tests, Testnet (12):

10. An unfunded destination is reported as `destinationNotFound`.
11. A funded destination without the trustline is reported as `destinationNoTrustline`, and the source's sequence number and XLM balance do not change.
12. Five valid amounts (including one stroop and the maximum) and four valid memos (including 28 bytes with multibyte characters) get past validation.
13. The locally computed transaction hash equals the one Horizon reports.

Skipped (1):

14. A successful payment. It needs a funded Mainnet account holding SHx.

## Out of Scope for 0.0.7-dev

- **The success path.** The pass with a funded Mainnet account now covers four behaviors that wait for it: the trustline, the bootstrap, a non-zero SHx balance, and this payment.
- **The `outcomeUnknown` path.** No test simulates a network failure, and the line of `send()` that attaches the memo is not run by a test; the SDK calls it makes are.
- **Looking up a transaction by hash.** There is no helper yet for the caller who gets `outcomeUnknown`.
- **Path payments** (`0.0.8-dev`).
- **Muxed and contract destinations**, and payments through the Stellar Asset Contract.
- **Memos other than text.**
- **Fee control.** The SDK's default fee is used, and `tx_insufficient_fee` surfaces as `rejected`.
- **Concurrent sends from one account.** The collision surfaces as `tx_bad_seq`, reported as `rejected`.
- **Checking the source's SHx balance before sending.** The network answers with `op_underfunded`.

## Related

- [SHx Trustline](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/shx-trustline/README.md)
- [Balance Queries](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/balance-queries/README.md)
- [Bootstrap for SHx](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-2/account-bootstrap-helpers/README.md)
- [Flutter Dependency and Re-exports](../flutter-dependency-and-reexports/README.md): `KeyPair`, `Network`, and `StellarSDK`, which `send()` takes, come from the package barrel.

## Sources

- Stronghold, SHx documentation index: <https://docs.shx.stronghold.co/llms.txt>
- Stronghold, SHx page: <https://stronghold.co/shx>
- Stronghold, GitHub organization: <https://github.com/orgs/strongholdpay/repositories>
- Stellar documentation, send and receive payments: <https://developers.stellar.org/docs/tutorials/send-and-receive-payments/>
- Stellar documentation, payment result codes: <https://developers.stellar.org/docs/data/apis/horizon/api-reference/errors/result-codes/operation-specific/payment>
- Stellar documentation, transaction result codes: <https://developers.stellar.org/docs/data/apis/horizon/api-reference/errors/result-codes/transactions>
- Stellar documentation, transaction failed: <https://developers.stellar.org/docs/data/apis/horizon/api-reference/errors/http-status-codes/horizon-specific/transaction-failed>
- Stellar documentation, error handling: <https://developers.stellar.org/docs/data/apis/horizon/api-reference/errors/error-handling>
- Stellar documentation, list of operations: <https://developers.stellar.org/docs/learn/fundamentals/transactions/list-of-operations>
- Stellar documentation, assets and amount precision: <https://developers.stellar.org/docs/learn/fundamentals/stellar-data-structures/assets>
- Stellar documentation, verify trustlines: <https://developers.stellar.org/docs/build/guides/basics/verify-trustlines>
- Stellar documentation, the Stellar Asset Contract: <https://developers.stellar.org/docs/tokens/stellar-asset-contract>
- Stellar documentation, the account object and its flags: <https://developers.stellar.org/docs/data/apis/horizon/api-reference/resources/accounts/object>
- `stellar_flutter_sdk`, `Memo`: <https://pub.dev/documentation/stellar_flutter_sdk/latest/stellar_flutter_sdk/Memo-class.html>
- `stellar_flutter_sdk`, `SubmitTransactionResponse`: <https://pub.dev/documentation/stellar_flutter_sdk/latest/stellar_flutter_sdk/SubmitTransactionResponse-class.html>
