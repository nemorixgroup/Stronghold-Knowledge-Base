# Stronghold / SHx Knowledge Base
 
> A comprehensive, in-depth technical guide to Stronghold and the SHx token ecosystem, covering the SHx asset on Stellar, its governance mechanics, its on-chain escrow infrastructure, StrongholdNET, and the development ecosystem for building on top of it.

All content in this repository is sourced exclusively from official Stronghold/SHx documentation, official Stellar network data (Horizon, Soroban RPC, on-chain contract inspection), and verified on-chain findings. See the [Sources](#official-resources) section for references. Where information could not be independently verified, it is explicitly flagged rather than assumed.
 
## About This Repository
 
This repository is a structured knowledge base built for developers, researchers, and engineers who want to understand Stronghold and the SHx token at a deep technical level, including several details not clearly documented anywhere else, uncovered through direct on-chain research (see Modules 03 and 04).
 
The content is organized in progressive modules, starting from foundational concepts and advancing toward hands-on development. Each module is self-contained and can be studied independently, but reading them in order is recommended for a complete understanding.
 
**Who is this for?**
 
- Developers exploring the SHx token ecosystem for the first time
- Engineers evaluating SHx/Stellar for payments, settlement, or governance-participation projects
- Blockchain enthusiasts who want to understand how Stronghold differs from ecosystems that are themselves independent blockchains
- Flutter/Dart developers interested in building on SHx (see Module 06 and [`stronghold_flutter_sdk`](https://github.com/nemorixgroup/stronghold-flutter-sdk))

**Languages:** English (primary). Spanish content delivered first during development, per project workflow.
 
## What is Stronghold?
 
Stronghold is a fintech infrastructure company (founded 2015, launched 2017), not a blockchain. It operates **StrongholdNET**, a proprietary payments platform that bridges traditional bank rails (ACH, ISO 20022) with distributed ledgers, and it issues **SHx**, a utility token used for settlement, merchant rewards, liquidity, and on-chain governance.
 
This is the single most important distinction to internalize before going further: **SHx does not have its own chain.** It is an asset issued on the **Stellar network**, with mirrored representations on Ethereum, Solana, and the XRP Ledger for exchange liquidity. Everything technically buildable today, the asset itself, its governance mechanism, and its on-chain escrow infrastructure, lives on Stellar.
 
## Repository Structure
 
```
stronghold-knowledge-base/
- README.md                          <- You are here (General Index)
- module-01-foundations/
    - README.md                      <- Company history, SHx origin, mission, EDP grant program
- module-02-shx-token/
    - README.md                      <- SHx as a Stellar asset: issuer, supply, decimals, trustlines
- module-03-governance/
    - README.md                      <- Governance Rules, ManageData voting, proposal history
- module-04-escrow-contract/
    - README.md                      <- The 60B SHx Soroban escrow contract (verified on-chain spec)
- module-05-strongholdnet/
    - README.md                      <- StrongholdNET, Stronghold Pay API, SHx Bridge (Axelar), open-access roadmap
- module-06-development/
    - README.md                      <- SDK landscape, the mobile-SDK gap, Flutter integration plan
- module-07-ecosystem/
    - README.md                      <- Exchanges, use cases, the EDP grant, roadmap
```
 
## Modules
 
| #  | Module | Topics |
|----|--------|--------|
| 01 | [Foundations & History](module-01-foundations/README.md) | Company origin, founders, IBM as first client, SHx launch (2018), EDP mandate |
| 02 | [The SHx Token](module-02-shx-token/README.md) | Asset code and issuer, 7 decimals, 100B fixed supply, trustlines, the Soroban SAC wrapper |
| 03 | [Governance](module-03-governance/README.md) | Governance Rules document, `ManageData`-based voting, proposal history (#1-#7), Snapshot for Ethereum |
| 04 | [The Escrow Contract](module-04-escrow-contract/README.md) | 60B SHx / 5-year lock ladder, verified Soroban contract spec (`lock`, `unlock`, `extend_ttl`), source-verification gap |
| 05 | [StrongholdNET & Bridges](module-05-strongholdnet/README.md) | ACH/ISO 20022 rails, Stronghold Pay API, SHx Bridge via Axelar, the still-unreleased open-access API console |
| 06 | [Development Ecosystem](module-06-development/README.md) | `stellar_flutter_sdk` as the composition base, the mobile-SDK gap, architecture plan for `stronghold_flutter_sdk` |
| 07 | [Ecosystem & Grants](module-07-ecosystem/README.md) | Exchange listings, community size, the EDP Ecosystem Grants program, our application |
 
## Key Technical Facts (Quick Reference)
 
| Property | Value |
|---|---|
| SHx is a native blockchain | **No**, it is a Stellar asset, not an independent network |
| Asset code / Issuer | `SHX` / `GDSTRSHXHGJ7ZIVRBXEYE5Q74XUVCUSEKEBR7UCHEUUEK72N7I7KJ6JH` |
| Decimals | 7 |
| Total supply | 100,000,000,000 SHx (fixed, no ICO/TGE/IEO) |
| Governance voting mechanism | Classic Stellar `ManageData` operations, not a Soroban contract |
| Soroban Asset Contract | `CCKCKCPHYVXQD4NECBFJTFSCU2AMSJGCNG4O6K4JVRE2BLPR7WNDBQIQ`, auto-generated SEP-41 wrapper, standard token interface only |
| Escrow contract (60B SHx, 5-year lock) | `CCA5HAZCPEYXD7JBKAJCVUZUXAK7V5ZFU3QMJO33OJH2OHL3OGLS2P7M`, custom code, spec verified on-chain |
| Escrow contract source code | Not publicly verified (no Contract Source Validation SEP attestation found) |
| Mirrored networks | Ethereum (ERC-20), Solana, XRP Ledger; liquidity/exchange access only |
| Cross-chain bridge | SHx Bridge (Stellar to Ethereum) via Axelar interoperability protocol |
| Official Flutter/Dart SDK | None exists today (for SHx or for Stronghold Pay) |
| Last official mobile SDK | `react-native-sdk` (Stronghold Pay only), unmaintained since Dec 2022 |
| Public Soroban contract source repos | None found in the `strongholdpay` GitHub org |
 
## Why This Knowledge Base Exists
 
Unlike Avalanche, Hedera, or XRPL, Stronghold does not publish a single consolidated technical reference for its on-chain infrastructure. Governance mechanics, the escrow contract, and the SAC wrapper are each documented (if at all) in different places, and some details required direct on-chain verification rather than reading documentation. This repository consolidates that research into one place, distinguishing clearly between:
 
- **Documented and confirmed** (governance rules, voting format, token supply)
- **Verified directly on-chain** (the SAC wrapper's actual functions, the escrow contract's real spec, both pulled via `stellar contract inspect` against Mainnet, not assumed from marketing copy)
- **Publicly claimed but unverifiable** (e.g. escrow contract source code, described as "fully visible" but not attested via any source-verification mechanism)

## Official Resources
 
### Core
 
| Resource | URL |
|---|---|
| Company website | <https://stronghold.co> |
| SHx token page | <https://stronghold.co/shx> |
| SHx governance docs | <https://docs.shx.stronghold.co> |
| Ecosystem Development Program (EDP) | <https://docs.shx.stronghold.co/ecosystem/edp> |
| Stronghold Pay API docs | <https://docs.strongholdpay.com> |
| SHx Bridge | <https://gateway.stronghold.co/bridge> |
 
### On-Chain / Developer Tools
 
| Resource | URL |
|---|---|
| Stellar Lab (Contract Explorer, requires Mainnet RPC URL) | <https://lab.stellar.org/smart-contracts/contract-explorer> |
| Stellar Expert (SHx asset) | <https://stellar.expert/explorer/public/asset/SHX-GDSTRSHXHGJ7ZIVRBXEYE5Q74XUVCUSEKEBR7UCHEUUEK72N7I7KJ6JH> |
| SHx Voting Tool | <https://vote.stronghold.co> |
| SHx Ethereum governance (Snapshot) | <https://snapshot.box/#/s:strongholdco.eth> |
 
### GitHub
 
| Organization | URL |
|---|---|
| Stronghold (official) | <https://github.com/strongholdpay> |
| Stronghold `react-native-sdk` (unmaintained) | <https://github.com/strongholdpay/react-native-sdk> |
 
### Community & News
 
| Resource | URL |
|---|---|
| Stronghold News / Blog | <https://stronghold.co/learn-categories/news> |
| SHx Discord | via <https://stronghold.co/social> |
 
## Development on Stronghold/SHx
 
Because SHx runs on Stellar rather than its own chain, the base SDK dependency is the actively-maintained, Stellar Public Goods Program-funded [`stellar_flutter_sdk`](https://github.com/Soneso/stellar_flutter_sdk) (Soneso, MIT license). `stronghold_flutter_sdk` is designed as a **composition layer** on top of it, not a fork, adding SHx-specific asset helpers, governance-voting helpers, and escrow-contract bindings. See Module 06 for the full architecture rationale.
 
## About This Guide
 
This knowledge base was built through official Stronghold/SHx documentation, official Stellar network data, and direct on-chain verification, including inspecting both the SHx Soroban Asset Contract and the 60B SHx escrow contract live on Mainnet to confirm their real function signatures, rather than relying on secondary descriptions. It is being built alongside [`stronghold_flutter_sdk`](https://github.com/nemorixgroup/stronghold-flutter-sdk), the first native Flutter/Dart SDK for the SHx ecosystem, and the supporting technical documentation for Stronghold's Ecosystem Development Program (EDP) grant application.
 
The goal is to provide the most clear, honest, and technically accurate resource available for understanding Stronghold and SHx, from the inside out, and without repeating unverified marketing claims as fact.
 
## Related Repositories
 
| Repository | Description |
|---|---|
| [Hedera-Knowledge-Base](https://github.com/nemorixgroup/Hedera-Knowledge-Base) | In-depth guide to the Hedera network |
| [XRPL-Knowledge-Base](https://github.com/nemorixgroup/XRPL-Knowledge-Base) | In-depth guide to the XRP Ledger |
| [Avalanche-Knowledge-Base](https://github.com/nemorixgroup/Avalanche-Knowledge-Base) | In-depth guide to the Avalanche network |
| [Stellar-Knowledge-Base](https://github.com/nemorixgroup/Stellar-Knowledge-Base) | In-depth guide to the Stellar network (SHx's underlying chain) |
 
## Contributing
 
This is a personal knowledge base maintained under the Nemorix Group organization. Contributions, corrections, and suggestions are welcome via issues or pull requests.
 
## License
 
MIT License, see [LICENSE](LICENSE) for details.
 
## Support This Project
 
If this project is useful to you or your team, consider supporting its development. Every contribution helps cover infrastructure, documentation, and the time invested in building and maintaining this open source resource for the SHx ecosystem. Thank you!
 
[![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Support-FFDD00?logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/nemorixgroupllc)
[![Sponsor](https://img.shields.io/badge/Sponsor-GitHub-EA4AAA?logo=github-sponsors&logoColor=white)](https://github.com/sponsors/nemorixgroup)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support-FF5E5B?logo=ko-fi&logoColor=white)](https://ko-fi.com/nemorixgroupllc)
 
---
 
*Last updated: July 2026*  
*Maintained by: [Miguel Fagundez](https://github.com/miguelfagundez) & [Nemorix Group, LLC](https://github.com/nemorixgroup)*  
