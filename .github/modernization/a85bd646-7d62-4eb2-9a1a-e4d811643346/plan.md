## Goal and target

Retarget the small synthetic pricing sample from .NET 8 to .NET 10 without changing behavior. Add **exactly one** regression test before retargeting. Record exact final build and test results with marker `acceptance-20261025051138-4c2f94-windows-validated`.

## Approach

The assessment records a green baseline: build succeeded with zero warnings and errors; TRX reported **9 executed, 9 passed, 0 failed, 0 skipped**. First pin receipt behavior on unchanged `net8.0`, then change the existing project's target framework, then validate and report. Each task uses an independently runnable full build and full-suite test command from the repository root. Both commands permit restore and build as needed; neither relies on `--no-build` or `--no-restore`. Compare TRX test identities and counts after each task. No lint gate is configured.

Only tracked changes to the sample files and `.github/modernization/a85bd646-7d62-4eb2-9a1a-e4d811643346/` are planned. `TestResults/validation` is the repository's already-ignored test-output location, not a tracked change.

## Decisions

- **Target:** Set `Pricing.csproj` from `net8.0` to `net10.0` in place. Source: assessment §§ Stack inventory and Recommended approach; user goal. The assessment confirms an installed .NET 10 SDK and runtime.
- **Test:** Add one test for exact invariant-culture receipt text with a trimmed lowercase valid discount code while the current culture is non-English; restore the original culture in `finally`. Source: assessment § Coverage and characterization candidates and `PriceCalculator.cs:43-69`.
- **Dependencies:** Keep the three direct test-package versions pinned unless an actual .NET 10 gate failure establishes a blocker; do not undertake a general upgrade. Source: assessment § Dependencies and vulnerabilities.
- **Discovery threshold:** The nine baseline cases must remain discovered and executed; after the one new test, require **10 executed, 10 passed, 0 failed, 0 skipped**, not merely a successful `dotnet test` exit. Source: assessment §§ Build and CI and Test baseline; user goal.

## Assumptions

- The assessment's green .NET 8 baseline and installed .NET 10 toolchain remain available.
- The project has no CI, deployment, service dependency, or configured lint gate, as recorded in the assessment.
- If an incompatible dependency or behavior change proves necessary, stop and obtain a decision rather than silently broadening this plan.

## Out of scope

Deployment, workflow or container edits, secrets, pushing, PR creation, broad dependency refreshes, additional tests, and changes outside the sample and run artifact folder.

## Tasks

### T001: Add one .NET 8 behavior characterization test
- Depends on: -
- Skills: test-baseline-and-characterization, build-fix-loop
- Validation:
  - build: `dotnet build Pricing.csproj --nologo`
  - test: `dotnet test Pricing.csproj --logger trx --results-directory TestResults/validation --nologo`
- Characterization: On unchanged `net8.0`, add exactly one `PriceCalculatorTests.cs` test asserting the complete receipt string for a trimmed lowercase valid code under a non-English current culture; restore the prior culture in `finally`.
- Risk: low (culture-sensitive test setup could affect other tests); mitigate by restoring culture and confirming the full TRX suite.
- Done when: The sole new test passes on .NET 8, the full build is green, and TRX shows 10 executed and passed with the original nine retained.

Keep production code and the project target unchanged. Record the test name and validation evidence in this task's progress note.

### T002: Retarget Pricing in place to .NET 10
- Depends on: T001
- Skills: build-fix-loop, test-baseline-and-characterization
- Validation:
  - build: `dotnet build Pricing.csproj --nologo`
  - test: `dotnet test Pricing.csproj --logger trx --results-directory TestResults/validation --nologo`
- Characterization: Retain T001's receipt test and all nine original cases; compare their TRX identities and outcomes with the .NET 8 run.
- Risk: medium (older test packages could affect .NET 10 discovery, or formatting could change); mitigate with the full gates, TRX identity/count comparison, and review after the task.
- Done when: `Pricing.csproj` targets `net10.0`, the full build is green, and TRX shows the same 10 cases executed and passed with zero failures or skips.

Change the target framework only. Keep dependencies pinned unless a demonstrated blocker requires a separately decided, narrowly scoped fix; do not weaken tests or validation.

### T003: Final validation and report
- Depends on: T001, T002
- Skills: modernization-report, test-baseline-and-characterization
- Validation:
  - build: `dotnet build Pricing.csproj --nologo`
  - test: `dotnet test Pricing.csproj --logger trx --results-directory TestResults/validation --nologo`
- Characterization: Verify that the one added test and all nine baseline cases remain discovered, executed, and passing on .NET 10.
- Risk: low (a report could misstate final gate results); mitigate by taking exit codes, warnings, durations, and test counts from the final command outputs and TRX, and reconciling any completion-gate rerun.
- Done when: Final full gates are green with 10/10 tests passed and none failed or skipped, and `final-report.md` records the **exact observed** build/test commands and results, baseline comparison, task outcomes, and marker `acceptance-20261025051138-4c2f94-windows-validated`.

Write `.github/modernization/a85bd646-7d62-4eb2-9a1a-e4d811643346/final-report.md` using the modernization-report outline. Distinguish measured final results from expectations; document unchanged dependencies, relevant scope exclusions, and any remaining risk.

## Risks

The principal upgrade risk is silent test-discovery loss, so every task checks machine-readable counts and T002 compares test identities. Receipt formatting is pinned before retargeting. No high-risk task is planned; an unexpected compatibility fix or behavior change is a stop condition, not permission to expand scope.
