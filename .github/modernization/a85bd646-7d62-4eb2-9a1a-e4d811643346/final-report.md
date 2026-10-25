# Modernization report: acceptance-20261025051138-4c2f94 windows modernization

Outcome: SUCCEEDED - 2 of 3 tasks checkpoint-completed (T003 report written, checkpoint pending); build GREEN; tests 10/10 passing (baseline 9/9).

## Summary

- Goal: Retarget the synthetic pricing sample to .NET 10 without changing observed behavior, add one regression test, and record exact gates. Marker: `acceptance-20261025051138-4c2f94-windows-validated`.
- Scenario: not specified in run state; scope: this sample and its run artifacts only. Strategy: in-place retarget of the single project, preceded by a behavior-characterization test ([assessment](assessment.md), [plan](plan.md)).
- Branch: `acceptance-20261025051138-4c2f94-windows-modernization` based on `main`; two existing checkpoint commits. T003 is in progress in the run ledger, so its checkpoint commit is **pending**, not claimed here.
- Changed sample files: `Pricing.csproj` and `PriceCalculatorTests.cs`. Added run artifacts: `assessment.md`, `plan.md`, `progress/T001.md`, `progress/T002.md`, and this report. `PriceCalculator.cs` was not edited. Source: `git diff --name-status main...HEAD` before this report was written, plus this report.

## Versions

| Component | Before | After | Support/LTS status and source |
|---|---|---|---|
| Project target framework | `net8.0` | `net10.0` | .NET 8 LTS maintenance support through 2026-11-10; .NET 10 LTS active support through 2028-11-14, per [Microsoft .NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core), checked 2026-10-25 in the [assessment](assessment.md#support-status). |
| Locally installed SDK / runtime | SDK 10.0.402 used to build `net8.0`; installed .NET 8 runtime 8.0.31 | SDK 10.0.402 available; installed .NET 10 runtime 10.0.13 | Environment inventory in [assessment](assessment.md#stack-inventory). The installed .NET 8 runtime was one patch behind the 8.0.32 listed by the support policy on 2026-10-25; no SDK pin is present. |
| `Microsoft.NET.Test.Sdk` | 17.11.1 | 17.11.1 | [NuGet package page](https://www.nuget.org/packages/Microsoft.NET.Test.Sdk/17.11.1), checked 2026-10-25 in the [assessment](assessment.md#support-status); individual support end date unknown. |
| `xunit` | 2.9.2 | 2.9.2 | [NuGet package page](https://www.nuget.org/packages/xunit/2.9.2), checked 2026-10-25 in the [assessment](assessment.md#support-status); individual support end date unknown. |
| `xunit.runner.visualstudio` | 2.8.2 | 2.8.2 | [NuGet package page](https://www.nuget.org/packages/xunit.runner.visualstudio/2.8.2), checked 2026-10-25 in the [assessment](assessment.md#support-status); individual support end date unknown. |

## Tasks

| Key | Title | Status at report writing | Checkpoint commit | Result |
|---|---|---|---|---|
| T001 | Adding .NET 8 behavior characterization test | Completed | `a55817bb4873f9d988cb0359646a3b03e7c4d302` | Added one receipt regression test on `net8.0`; 10/10 passed ([progress](progress/T001.md)). |
| T002 | Retargeting Pricing in place to .NET 10 | Completed | `814cef496cb960f5b11c65c789b1a6b90f311179` | Changed only the target framework; same ten test identities passed ([progress](progress/T002.md)). |
| T003 | Running final validation and report | In progress; report written | Pending checkpoint | Authoritative final gates passed; task completion/checkpoint occurs after this report. |

No failed or skipped tasks are recorded in the run ledger. Decisions: characterize the original receipt before migration, then retarget in place; retain pinned test dependencies because no gate demonstrated an incompatibility ([plan](plan.md#decisions), [T001](progress/T001.md#decisions), [T002](progress/T002.md#decisions)). Do not expand to application rewrites, dependency refreshes, or deployment.

## Validation

| Gate | Command | Baseline | Final authoritative result |
|---|---|---|---|
| Build | Baseline: `dotnet restore Pricing.csproj --nologo`; `dotnet build Pricing.csproj --no-restore --nologo`. Final: `dotnet build Pricing.csproj --nologo` | Both exit 0; build 0 warnings, 0 errors; build wall 3,561 ms after restore ([assessment](assessment.md#build-and-ci)). | Exit 0; 0 warnings, 0 errors; wall 1.400 s (final BuildValidator measurement). |
| Full tests | `dotnet test Pricing.csproj --logger trx --results-directory TestResults/baseline --nologo` (baseline); `dotnet test Pricing.csproj --logger trx --results-directory TestResults/validation --nologo` (final) | Exit 0; TRX 9 executed, 9 passed, 0 failed, 0 skipped; wall 3,681 ms ([assessment](assessment.md#build-and-ci)). | Exit 0; wall 2.174 s; `TestResults/validation/ContainerUser_RUN-A85BD6467D6_2026-10-25_06_18_20_net10.0.trx`: 10 executed, 10 passed, 0 failed, 0 skipped (final BuildValidator measurement and TRX). The same nine baseline identities and sole added identity remain present. |
| Lint | None configured | Not applicable | Not applicable |

- Added characterization/regression test: `Acceptance.Pricing.PriceCalculatorTests.ReceiptUsesInvariantFormattingAndNormalizesCouponUnderFrenchCulture` in `PriceCalculatorTests.cs`. On unchanged .NET 8 it passed along with all nine baseline tests, then all ten passed after retargeting ([T001](progress/T001.md#validation), [T002](progress/T002.md#validation)). There were no test-discovery losses.
- Approved test exclusions: none; final TRX reports no skipped tests. The final gates were measured after the last code change. Test-result files under `TestResults/` are ignored output, not tracked changes.

## Dependencies and security

| Package | Before | After | Change and reason |
|---|---|---|---|
| `Microsoft.NET.Test.Sdk` | 17.11.1 | 17.11.1 | None; full .NET 10 gates passed. |
| `xunit` | 2.9.2 | 2.9.2 | None; full .NET 10 gates passed. |
| `xunit.runner.visualstudio` | 2.8.2 | 2.8.2 | None; all ten tests discovered and passed. |

No packages were added, removed, replaced, or upgraded (`git diff main...HEAD -- '*.csproj'` and [T002](progress/T002.md)). Baseline `dotnet list Pricing.csproj package --vulnerable --include-transitive` reported critical 0, high 0, moderate 0, low 0 from configured NuGet sources ([assessment](assessment.md#dependencies-and-vulnerabilities)); **no post-retarget vulnerability scan was recorded**, so post-change severity counts and remaining findings are unknown, not asserted to be zero. No vulnerability fix is claimed. Package-specific support lifetimes and independently verified transitive compatibility remain unknown; successful local build and test are compatibility evidence, not a security audit.

## Behavior and compatibility changes

| Change | Impact | Decision and test evidence |
|---|---|---|
| Target framework `net8.0` to `net10.0` | .NET 10 runtime required to run the retargeted tests; production code was not edited. | In-place target change ([plan](plan.md#decisions)); baseline nine identities plus the new receipt test passed on .NET 10. Behavior beyond tested paths is not verified. |
| Receipt text and coupon normalization | No observed change; invariant decimal formatting and trimmed, case-normalized coupon are pinned under `fr-FR`. | Characterize existing behavior before retargeting ([T001](progress/T001.md)); `ReceiptUsesInvariantFormattingAndNormalizesCouponUnderFrenchCulture` passed on both frameworks ([T002](progress/T002.md)). |

This sample has no HTTP routes, serialization surface, ports, or application configuration in the [assessment](assessment.md#stack-inventory); no such compatibility migration was made.

## CI, deployment and configuration

- Workflow files changed: none (`git diff main...HEAD -- .github/workflows 'Dockerfile*' '*.yml'` returned no changes). No CI workflow exists in the assessed repository. No workflow-change confirmation is needed for this diff.
- Runtime prerequisites: a .NET 10 runtime is required to execute the retargeted project; SDK 10.0.402 and runtime 10.0.13 were available in the assessed local environment. Production hosting requirements are unknown because this is a test sample, not a deployable application.
- Configuration or environment variable names added/renamed: none. Container, deployment, infrastructure, secrets, and service-connection changes: none. No deployment was performed.

## Risks and follow-ups

| Item | Severity | Recommendation |
|---|---|---|
| No post-retarget vulnerability audit; pinned direct and transitive package maintenance/support dates not fully established | Low / security status unknown | Review dependency lifecycle and run `dotnet list Pricing.csproj package --vulnerable --include-transitive` against approved sources before treating the upgrade as security-cleared. |
| Tests cover ten known cases, not all possible arithmetic or inputs; no coverage measurement | Low | If broader behavior assurance becomes necessary, seek approval for additional characterization (e.g., overflow/null inputs) rather than expanding this task silently. |
| T003 checkpoint and final review not yet recorded at report-writing time | Process pending | Complete the authorized task gate/checkpoint and review the resulting diff before pushing from Futuristic. |

## Open questions

- No blocking question is recorded. Whether to refresh the pinned test packages or add broader characterization is future scope, not a prerequisite for the measured build/test gates.

## How to verify locally

From the repository root with a .NET 10 SDK/runtime available:

```powershell
dotnet build Pricing.csproj --nologo
dotnet test Pricing.csproj --logger trx --results-directory TestResults/validation --nologo
```

There is no application run command or health URL: `Pricing.csproj` is a test project ([assessment](assessment.md#stack-inventory)). Review the changes and push them **from Futuristic** after checkpoint and review; do not deploy, add secrets, or open a PR from this workspace.

## Artifacts

- [Assessment](assessment.md)
- [Approved plan](plan.md)
- [T001 progress](progress/T001.md)
- [T002 progress](progress/T002.md)
- [T003 final validation progress](progress/T003.md)
