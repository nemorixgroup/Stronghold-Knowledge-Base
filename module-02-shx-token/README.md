# Module 02: The SHx Token

## SHx as a Stellar Asset

| Property | Value |
|---|---|
| Asset code | `SHX` |
| Issuer | `GDSTRSHXHGJ7ZIVRBXEYE5Q74XUVCUSEKEBR7UCHEUUEK72N7I7KJ6JH` |
| Network | Stellar Public Global Network |
| Decimals | 7 |
| Total supply | 100,000,000,000 SHx, fixed, with no additional issuance mechanism |

On Stellar, every non-native asset (anything that isn't XLM) is always identified by the pair `(code, issuer)`, never by the code alone. This is central to understanding why this SDK hardcodes the issuer: anyone can create an asset with the code `SHX` from a different issuing account, and it would be a completely distinct asset with no relation to the real SHx. The only way to confirm a `SHX` asset is "the" SHx is by verifying its issuer matches `GDSTRSHX...` exactly.

## Trustlines: The Mandatory Requirement to Receive SHx

Unlike blockchains where any account can receive any token without prior setup, Stellar requires an explicit opt-in step for every non-native asset. This mechanism is called a **trustline**.

**Mandatory flow for a new account to receive SHx:**
1. The account submits a `ChangeTrust` operation declaring it trusts `(SHX, GDSTRSHX...)`, up to an optional maximum limit.
2. Only after that can the account receive a `Payment` operation in SHx.

**Important side effect for UX:** each trustline consumes part of the account's *minimum balance reserve* on Stellar (currently 0.5 XLM per active trustline, on top of the account's base reserve). This means a user needs XLM already available in their account *before* they can receive SHx for the first time. Any onboarding flow built on top of this SDK has to account for this prerequisite; it is not optional and cannot be bypassed.

To remove a trustline (for example, if a user wants to stop holding SHx), the same `ChangeTrust` operation is submitted with a limit of `0`, but **this only succeeds if the account's SHx balance is already zero**. A trustline cannot be closed while it still holds funds.

## The Soroban Wrapper (SAC): What SHx Is Not

When Soroban went live on Mainnet (February 20, 2024), the protocol began automatically generating, for every existing classic asset, a contract called a **Stellar Asset Contract (SAC)**. This is not a Stronghold decision; it is protocol-level Stellar behavior, applied uniformly to all assets.

This was independently verified on-chain (`stellar contract inspect` against Mainnet, contract `CCKCKCPHYVXQD4NECBFJTFSCU2AMSJGCNG4O6K4JVRE2BLPR7WNDBQIQ`):

- **Contract creation date:** February 21, 2024, one day after Soroban's activation, matching exactly when the protocol began generating these wrappers.
- **Source code:** `stellar/rs-soroban-env`, protocol code, not Stronghold-authored.
- **Exposed functions:** the full SEP-41 standard (Stellar's conceptual equivalent to ERC-20): `balance`, `transfer`, `transfer_from`, `approve`, `allowance`, `mint`, `burn`, `burn_from`, `clawback`, `set_admin`, `set_authorized`, `decimals`, `name`, `symbol`. No governance, lock, or voting logic of any kind.

**Why this matters:** it is easy, and it was our own first assumption, to read this Contract ID, documented on Stronghold's own governance page, and assume it was a custom governance contract. It is not. It is a Stellar infrastructure detail, not a Stronghold product decision. Anyone building on SHx who assumes otherwise will waste time looking for voting functions that do not exist there.

## What Is Actually Needed to Work with SHx on Soroban

The SAC is useful, not irrelevant, in one specific case: if a **custom Soroban contract** (not a classic account) ever needs to receive or transfer SHx, that contract interacts with SHx through this SEP-41 wrapper, using the standard token interface. It is SHx's entry point into the smart contract world, even though it carries no logic of its own.

---

*Sources: [docs.shx.stronghold.co/shx/shx-governance-rules](https://docs.shx.stronghold.co/shx/shx-governance-rules),   
on-chain verification via `stellar contract inspect` against Mainnet (contract CCKCKCPHYVXQD4NECBFJTFSCU2AMSJGCNG4O6K4JVRE2BLPR7WNDBQIQ).*
