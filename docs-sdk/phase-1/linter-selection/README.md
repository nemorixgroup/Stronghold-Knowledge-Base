# Linter Selection

**Phase:** 1 - Architecture & Core Setup  
**Status:** ✅ Done (`0.0.1-dev`)  
**Implementation:** [`pubspec.yaml`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/pubspec.yaml), [`analysis_options.yaml`](https://github.com/nemorixgroup/stronghold-flutter-sdk/blob/master/analysis_options.yaml)

## What This Is

Which static analysis ruleset `stronghold_flutter_sdk` enforces via `dart analyze --fatal-infos`.

## The Question

The initial scaffold used `flutter_lints`, the default Flutter/Dart linter, since that is what `flutter create --template=package` configures out of the box. That choice was made before checking what the rest of the Nemorix Group SDK portfolio uses.

## Options Considered

1. **Keep `flutter_lints`.** This was already in place, already passing with 0 issues, and is a reasonable, widely-used default. Rejected on review, not because it was wrong on its own, but because it was inconsistent with every other Nemorix SDK.
2. **Switch to `very_good_analysis`.** Chosen, to match `avalanche_flutter_sdk`, `hedera_flutter_sdk`, and `xrpl_flutter_sdk`, all of which use this stricter ruleset. Consistency across the portfolio matters more here than either option's individual merits: a contributor moving between Nemorix SDKs should not have to re-learn a different lint standard each time.

## What Changed as a Result

Switching rulesets surfaced 94 issues that `flutter_lints` had not flagged, entirely lint-level (constructor ordering, missing dartdoc on public members, relative imports instead of `package:` imports, redundant local variable type annotations, unnecessary `break` statements in switch cases, TODO comment formatting), no behavioral changes. All 94 were resolved before Phase 1 closed.

A version-compatibility issue also surfaced during this change: `very_good_analysis ^10.0.0` requires Dart >=3.9.0, but this SDK's declared minimum is Dart >=3.8.0 (matching the CI environment's Flutter 3.32.0). The dependency was pinned to `very_good_analysis ^9.0.0` instead of raising the SDK's minimum supported Dart version to accommodate a linter package.

## Decision

`very_good_analysis ^9.0.0`, for portfolio-wide consistency, with the version pinned to respect the SDK's own stated minimum Dart SDK support rather than the other way around.

## Related

- [Composition, Not a Fork](https://github.com/nemorixgroup/Stronghold-Knowledge-Base/blob/main/docs-sdk/phase-1/composition-not-fork/README.md)
