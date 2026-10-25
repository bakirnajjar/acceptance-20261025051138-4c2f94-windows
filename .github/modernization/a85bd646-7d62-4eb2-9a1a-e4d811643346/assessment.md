# Assessment — acceptance-20261025051138-4c2f94-windows-validated

## Summary

**Observed:** This is a five-file, single-project .NET 8 synthetic pricing test sample. The unmodified baseline is GREEN: restore and build exit 0; the full test suite has **9 passed / 0 failed / 0 skipped**; no lint gate is configured. **Recommendation:** In-place retargeting to .NET 10 with one behavior-pinning regression test in a separate preceding task. Overall risk: **low**, chiefly test discovery or price-format behavior changing unnoticed. This is Phase 1 only; no source files were changed.

## Stack inventory

| Unit | Path | Kind / stack | Current version evidence | Candidate target |
|---|---|---|---|---|
| Pricing | `Pricing.csproj` | SDK-style C# test project; production types and xUnit tests compiled together | `<TargetFramework>net8.0</TargetFramework>` (`Pricing.csproj:1-9`); xUnit package versions at lines 12-14 | `net10.0` in the same project |

**Repository shape:** `git ls-files` returned five paths: two `.cs`, one `.csproj`, one `.md`, one `.gitignore`. Production code is `PriceCalculator.cs`; tests are `PriceCalculatorTests.cs`. The project graph has one leaf, `Pricing.csproj`, and **no ProjectReference**. No solution, SDK pin (`global.json`), central package props, NuGet configuration, lock file, wrapper, application executable, deployment unit, container file, CI workflow, Azure pipeline, Jenkinsfile, or deployment configuration was found in the tracked files or repository tree. No application setting names were found. Package management is NuGet via `PackageReference` (`Pricing.csproj:11-15`); observed restore source is public `https://api.nuget.org/v3/index.json`; no private feed was identified. `README.md:6-11` documents `dotnet build` and `dotnet test`. No project-specific lint is documented or configured.

**Environment:** `dotnet --info` reports Windows 10.0.26100 win-x64, SDK 10.0.402, MSBuild 18.9.15, host 10.0.13; installed SDKs 8.0.425, 10.0.401, 10.0.402 and installed .NETCore.App runtimes 8.0.31, 10.0.12, 10.0.13. There is no `global.json`, so the available SDK selected for the baseline is 10.0.402, building a `net8.0` target. The installed 8.0.31 runtime is below the officially listed latest patch 8.0.32; see support source below.

## Support status

| Fact | Value and evidence | Source URL | Checked on |
|---|---|---|---|
| Current target support | .NET 8 is LTS, in maintenance support until **2026-11-10**; latest listed patch **8.0.32**. Current target `net8.0` is shown at `Pricing.csproj:4`; installed runtime 8.0.31 from `dotnet --info`. It is **not yet out of support** on the check date. | https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core | 2026-10-25 |
| Recommended target support | .NET 10 is LTS, active until **2028-11-14**; latest listed patch **10.0.13**. Locally installed SDK 10.0.402 and runtime 10.0.13 from `dotnet --info`. | https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core | 2026-10-25 |
| Servicing | Within a supported .NET release, staying current with patch updates is required to qualify for servicing; the support policy lists 8.0.32 as current. | https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core | 2026-10-25 |
| Test dependencies | NuGet's package page for `Microsoft.NET.Test.Sdk` 17.11.1 computes net10.0 compatibility; xUnit 2.9.2 and Visual Studio runner 2.8.2 are published, with the runner describing .NET 6+ support. Their individual end-of-support dates are **unverified**: no package-specific lifecycle date was established. Retargeted test discovery still requires a real run. | https://www.nuget.org/packages/Microsoft.NET.Test.Sdk/17.11.1 ; https://www.nuget.org/packages/xunit/2.9.2 ; https://www.nuget.org/packages/xunit.runner.visualstudio/2.8.2 | 2026-10-25 |

## Dependencies and vulnerabilities

| Package | Current (`Pricing.csproj:12-14`) | Supports target? / maintenance evidence | Known vulnerabilities | Recommended action |
|---|---|---|---|---|
| Microsoft.NET.Test.Sdk | 17.11.1 | NuGet computes net10.0 compatibility; latest reported 18.11.0; package support end date unverified. | None reported by current audit. | Keep pinned initially; upgrade only if the .NET 10 build/test gate requires it. |
| xunit | 2.9.2 | xUnit package published on NuGet; target-specific validation pending; latest reported 2.9.3; support end date unverified. | None reported by current audit. | Preserve existing test framework unless needed. |
| xunit.runner.visualstudio | 2.8.2 | Publisher says runner supports .NET 6+; latest reported 4.0.1; support end date unverified. | None reported by current audit. | Confirm all nine tests are discovered after retargeting. |

`dotnet list Pricing.csproj package --include-transitive` exit 0 also lists Microsoft.CodeCoverage 17.11.1, Microsoft.TestPlatform.ObjectModel 17.11.1, Microsoft.TestPlatform.TestHost 17.11.1, Newtonsoft.Json 13.0.1, System.Reflection.Metadata 1.6.0, xunit.abstractions 2.0.3, xunit.analyzers 1.16.0, xunit.assert/core/extensibility.core/extensibility.execution 2.9.2. These are transitive test dependencies; their individual lifecycle dates and .NET 10 compatibility are **unverified**. `dotnet list Pricing.csproj package --outdated --include-transitive` exit 0 reports the three direct updates above and updates to every listed transitive dependency except xunit.abstractions; this is **not** a reason for an unscoped dependency refresh. `dotnet list Pricing.csproj package --vulnerable --include-transitive` exit 0 reports **no vulnerable packages** from configured NuGet sources (critical 0, high 0, moderate 0, low 0); no package, advisory, or fixed-version entries were returned. This is a scan result, not a guarantee of no vulnerabilities.

## Build and CI

| Command | Source | Exit code | Wall duration | Result |
|---|---|---:|---:|---|
| `dotnet restore Pricing.csproj --nologo` | Preparatory restore for complete baseline | 0 | 12,094 ms | Restored project. |
| `dotnet build Pricing.csproj --no-restore --nologo` | Full `dotnet build` README gate, with equivalent explicit project and already-completed restore | 0 | 3,561 ms | Build succeeded, 0 warnings, 0 errors; output `bin/Debug/net8.0/Pricing.dll`. |
| `dotnet test Pricing.csproj --logger trx --results-directory TestResults/baseline --nologo` | Full `dotnet test` README gate plus machine-readable logger | 0 | 3,681 ms | TRX: 9 total, 9 executed, 9 passed, 0 failed, 0 not executed, 0 error; console test duration 106 ms. |

Results source: `TestResults/baseline/ContainerUser_RUN-A85BD6467D6_2026-10-25_06_11_14_net8.0.trx`, under ignored `TestResults/` (`.gitignore:3`); build artifacts under ignored `bin/` and `obj/` (`.gitignore:1-2`). The skill's `.modernization-scratch/` path would require modifying `.git/info/exclude`; per run constraint, no ignore or scratch changes were made. Only this assessment is written as tracked artifact. No failing test IDs, first error lines, baseline failure classifications, or flaky suspects: **none**. A second test run is not necessary for a green baseline. There is no CI definition and no CI-pinned runtime in this repository.

**Proposed full validation gates:** `dotnet restore Pricing.csproj --nologo`; `dotnet build Pricing.csproj --no-restore --nologo`; `dotnet test Pricing.csproj --no-build --logger trx --results-directory TestResults/validation --nologo` (same default Debug configuration as baseline). Compare TRX counts and IDs; after adding one test, expect at least 10 tests and zero failures/skips. Lint: **not configured**; do not invent a narrowing replacement.

## Test baseline

Machine-readable TRX counters above are authoritative: **9/0/0 passed/failed/skipped**, with no failures to classify. The suite exercises subtotal, discount code normalization and unknown codes, 105-cent half-up discount, receipt text and quantity, and invalid quantities/prices (`PriceCalculatorTests.cs:9-47`). Tool environment and exact commands are recorded above. No external service is involved.

## Test services

`PriceCalculator.cs:1-71` and `PriceCalculatorTests.cs:1-49` contain in-memory pricing operations only; no database, broker, cache, HTTP, Testcontainers, or external connection is referenced. `docker version` connected to Docker Engine 29.8.2 (client 29.8.2); `DOCKER_HOST` and `TESTCONTAINERS_HOST_OVERRIDE` variable **names** are present, but neither Docker nor a port binding is required here. A remote engine would require its host override rather than localhost if future tests used containers. No test service connection is needed or missing; no credential values were inspected or recorded.

## Coverage and characterization candidates

**Observed:** All production paths are in `PriceCalculator.cs:9-69`; the existing tests provide examples but no coverage measurement. `LineItem` negative/zero quantity and negative price are covered, but its `Sku` value and very large arithmetic inputs are not asserted. `SubtotalCents` uses `int` arithmetic (`PriceCalculator.cs:33-40`), and `Total` computes integer-rounded discounts (`:50-55`); `Receipt` uses invariant formatting (`:57-69`). Proposed **single regression test before retargeting**: assert `Receipt`'s exact output for a trimmed lowercase valid discount code under a non-English current culture, pinning both code normalization and invariant price formatting against the unchanged code. Restore culture in a `finally` block; avoid locale-dependent assumptions elsewhere. Run full baseline gates after adding it, then retarget. Overflow and null inputs are potential additional characterization candidates only if later approved scope needs them.

## Blockers and hotspots

| Pattern / observed evidence | Files affected | Migration approach |
|---|---:|---|
| Legacy .NET blocker search (`System.Web`, `HttpContext.Current`, `ServiceContract`, `AppDomain.CreateDomain`, `BinaryFormatter`, `System.Drawing`, Registry/EventLog/WMI, COM/PInvoke, HintPath) over tracked C# and project files | 0 (`git grep -n -E ...`) | No framework migration bridge identified. |
| Integer cents, discount rounding, and invariant receipt rendering (`PriceCalculator.cs:33-69`) | 1 | Preserve behavior; characterize formatting before retargeting. |
| SDK-style combined production/test project, xUnit discovery (`Pricing.csproj:1-15`) | 1 | Retarget in place, confirm count and IDs after .NET 10 test run. |

## Risks

| Risk | Severity | Likelihood | Evidence and mitigation |
|---|---|---|---|
| Test runner/adapter discovers fewer tests after retarget | medium | low | Three older pinned test packages (`Pricing.csproj:12-14`); compare TRX discovered/executed count, not just command exit. |
| Currency text or rounding behavior changes | medium | low | Invariant receipt and integer arithmetic (`PriceCalculator.cs:33-69`); add one culture-sensitive regression test before changing target, retain nine existing tests. |
| Current .NET 8 runtime servicing level behind latest patch | low | medium | `dotnet --info` says 8.0.31; official policy lists 8.0.32 on check date. Use installed current 10.0.13 for target; avoid unrelated environment modifications during assessment. |
| Dependency support lifetime and transitive target compatibility not independently proven | low | low | Package lifecycle dates unverified; real .NET 10 build and all-tests gate is decisive. No broad package upgrade without evidence. |

## Recommended approach

**Recommendation (not yet implemented):** Single-project in-place upgrade, not strangler or rewrite: no external application, deployment, or project references exist. Keep NuGet dependencies pinned initially, add a regression test to characterize behavior on .NET 8, then change only target framework to `net10.0`, restoring/building/testing with full project gates and comparing machine-readable test results. Do not modify workflows, deploy, add secrets, push, or expand outside this sample. Final report should record exact results and the required validation marker.

## Open questions

None: the user specifies .NET 10, one regression test, preservation of behavior, and scope. Any unexpected incompatible package or requirement to change behavior would require a separate decision before broadening scope.

## Candidate tasks

1. **Behavior characterization:** add one regression test for invariant receipt under a non-English culture with normalized coupon, running the existing full build/test gates on .NET 8. Skills: `test-baseline-and-characterization`, `build-fix-loop`. Depends on this assessment.
2. **In-place target update:** set `Pricing.csproj` target to `net10.0`; run full restore/build/test, compare TRX tests (at least 10; no losses), retaining dependency versions unless a demonstrated blocker requires change. Skills: `build-fix-loop`, `test-baseline-and-characterization`. Depends on task 1.
3. **Final validation and report:** rerun the full gates, record exact exits, durations and TRX counters with marker `acceptance-20261025051138-4c2f94-windows-validated`; document dependency audit and any differences. Skills: `modernization-report`, `test-baseline-and-characterization`. Depends on task 2. No workflow/deployment task is in scope.
