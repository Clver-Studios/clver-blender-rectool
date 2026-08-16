# CLVERFILM --- PROMPT 1 FIXATION / OBSERVED LOGS & PROBLEMS

## Purpose

This document is a **diagnostic handoff for Claude**.

It records only:

-   observed terminal logs,
-   observed Minecraft in-game errors,
-   tests that were run,
-   test results,
-   discovered files and test fixtures,
-   operational difficulties,
-   current verification status,
-   and the expected outcome.

### Important instruction to Claude

**Do not use this document as a coding tutorial.**

Do not follow implementation instructions from this document.

Do not assume that the suggested cause, architecture, API usage, or fix
method in this report is authoritative.

**You own the implementation.**

Use your own engineering judgment, inspect the repository yourself, and
consult your own authoritative/current sources before deciding how to
fix anything.

This report intentionally does **not** prescribe code changes.

The goal is to give you the evidence and problems that need
investigation, while leaving the actual implementation entirely to you.

------------------------------------------------------------------------

# 1. PROJECT

Project:

``` text
ClverFilm
```

Location:

``` text
C:\Users\Maurice\Desktop\Clver\Projects\ClverFilm
```

Current task:

``` text
Prompt 1 — Foundation & Ground Truth
```

The current focus is **stabilizing and verifying Prompt 1/M0**.

Do not assume Prompt 2 or later feature work is part of this fixation.

------------------------------------------------------------------------

# 2. OVERALL OBSERVED STATUS

Current automated verification status:

``` text
Gate 1 normal verification       PASS
Gate 1 negative fixture          NOT YET EXECUTED

Gate 2 valid pack                PASS
Gate 2 invalid pack              PASS — correctly rejected

Gate 3 normal transform tests    PASS — 8/8
Gate 3 negative sign-flip test   PASS — correctly detected injected failure
```

M0 status:

``` text
M0 pack exists                   YES
M0 pack found                    YES
M0 installed for testing         YES
M0 script startup                ERROR
T00 successful execution         NOT YET VERIFIED
T01-T10 measurements             NOT YET COMPLETED
```

The biggest currently observed blocker is the Minecraft Script API
startup error described below.

------------------------------------------------------------------------

# 3. GATE 1 --- NORMAL VERIFICATION LOG

Command executed from:

``` text
C:\Users\Maurice\Desktop\Clver\Projects\ClverFilm
```

Command:

``` powershell
npm run verify
```

Observed output:

``` text
> clver@0.0.0 verify
> tsc -p runtime/jsconfig.json
```

No TypeScript error was printed.

Result:

``` text
PASS
```

------------------------------------------------------------------------

# 4. GATE 1 --- NEGATIVE FIXTURE

Fixture found:

``` text
tests\fixtures\gate1_hallucinated_method.js
```

This fixture exists specifically as a deliberately invalid
API/type-check test.

However, during this testing session:

**The negative Gate 1 fixture was not actually executed and its failure
output was not captured.**

Therefore the current evidence is:

``` text
Gate 1 positive test = PASS
Gate 1 negative test = NOT YET VERIFIED
```

Do not assume the negative gate works solely because the fixture exists.

------------------------------------------------------------------------

# 5. GATE 2 --- VALIDATOR DISCOVERY

Files found:

``` text
schemas\blockception-v1.21.73-0\SCHEMAS_MANIFEST.txt

tools\validate_pack.py

tools\vendor_schemas.py
```

The validator was investigated from PowerShell.

------------------------------------------------------------------------

# 6. GATE 2 --- HELP COMMAND DIFFICULTY

Attempt:

``` powershell
python tools\validate_pack.py --help
```

Observed:

``` text
Not a directory: --help
```

The same was attempted from the `tools` directory:

``` powershell
cd "C:\Users\Maurice\Desktop\Clver\Projects\ClverFilm\tools"

python validate_pack.py --help
```

Observed:

``` text
Not a directory: --help
```

Running without arguments:

``` powershell
python validate_pack.py
```

produced:

``` text
Gate 2 - schema-validate a compiled pack in memory, before a byte is written.

The compiler returns the whole pack as {path: text}. This validates that map
against vendored, tag-pinned schemas and returns problems. Nothing touches disk
here, so an invalid pack is never partially written over the operator's world.

`jsonschema` is a dev and test dependency only. This module lives in tools/ and
is never imported by clver/src/clver/, which stays stdlib-only.

An unmapped JSON file is reported, not skipped. A validator that quietly passes
files it does not understand is the same failure mode as an engine that fails
silently, and it is not a gate.

Run against a directory:  python tools/validate_pack.py m0/behavior_packs/clver_m0

usage: python tools/validate_pack.py <pack-dir>
```

Observed issue:

``` text
--help is not behaving as a normal help invocation.
```

This is a usability observation only.

------------------------------------------------------------------------

# 7. GATE 2 --- VALID M0 PACK

Command:

``` powershell
python tools\validate_pack.py m0\behavior_packs\clver_m0
```

Observed result:

``` text
PASS - no problems.
```

Result:

``` text
Gate 2 valid-pack test = PASS
```

------------------------------------------------------------------------

# 8. GATE 2 --- INVALID PACK TEST

Fixture discovered:

``` text
tests\fixtures\gate2_wrong_pack
```

Contents:

``` text
manifest.json

cameras\presets\bad.json
```

Command:

``` powershell
python tools\validate_pack.py tests\fixtures\gate2_wrong_pack
```

Observed output:

``` text
Gate 2: 2 files from tests\fixtures\gate2_wrong_pack against blockception-v1.21.73-0
  UNKNOWN-KEY: cameras/presets/bad.json: at minecraft:camera_preset/extend_player_rendring: not a documented field (allowlist from S-CAMDOC). The vendored schema does not check this object.
  SCHEMA: manifest.json: at header/uuid: 'not-a-uuid' does not match '^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$' [general/manifest/manifest.2.json]
FAIL - 2 problem(s).
```

Result:

``` text
Gate 2 negative test = PASS
```

The validator correctly detected two problems:

1.  An undocumented key:

``` text
extend_player_rendring
```

2.  An invalid UUID:

``` text
not-a-uuid
```

This is a deliberate negative fixture, so the `FAIL` output is the
expected result of the test.

------------------------------------------------------------------------

# 9. GATE 3 --- NORMAL TRANSFORM TEST

Command:

``` powershell
python tests\test_transforms.py
```

Observed output:

``` text
PASS  test_axis_map_preserves_vector_length_for_every_candidate
PASS  test_camera_and_object_forward_are_different_axes
PASS  test_convention_parameters_are_required_and_validated
PASS  test_every_axis_map_candidate_is_a_valid_basis
PASS  test_feeding_an_armature_matrix_to_the_camera_function_raises
PASS  test_property_over_2000_random_rotations
PASS  test_roundtrip_closure_over_every_unk_convention
PASS  test_yaw_is_reported_unrecoverable_at_the_poles

8/8 passed
```

Result:

``` text
Gate 3 normal test = PASS
```

------------------------------------------------------------------------

# 10. GATE 3 --- DELIBERATE SIGN-FLIP TEST

Fixture:

``` text
tests\fixtures\gate3_sign_flip.py
```

Command:

``` powershell
python tests\fixtures\gate3_sign_flip.py
```

Observed output:

``` text
Gate 3 with a seeded sign flip (x component negated):
PASS  test_axis_map_preserves_vector_length_for_every_candidate
PASS  test_camera_and_object_forward_are_different_axes
PASS  test_convention_parameters_are_required_and_validated
PASS  test_every_axis_map_candidate_is_a_valid_basis
PASS  test_feeding_an_armature_matrix_to_the_camera_function_raises
FAIL  test_property_over_2000_random_rotations
      random closure failed at yaw=-219.7646726876001 pitch=69.18836607678325 [yaw_sign=-1 offset=0.0 pitch_sign=-1]
FAIL  test_roundtrip_closure_over_every_unk_convention
      yaw closure failed: authored 166.73842552987617, recovered -166.73842552987617 [yaw_sign=-1 offset=0.0 pitch_sign=-1]
PASS  test_yaw_is_reported_unrecoverable_at_the_poles

6/8 passed
```

Result:

``` text
Gate 3 negative test = PASS
```

Reason:

The fixture intentionally injects a sign error.

The property tests detected that error.

The `FAIL` lines are therefore expected evidence that the negative test
is effective.

------------------------------------------------------------------------

# 11. TEST FILE INVENTORY

The following files were found:

``` text
tests\test_transforms.py

tests\fixtures\gate1_hallucinated_method.js

tests\fixtures\gate3_sign_flip.py

tests\fixtures\jsconfig.wrong.json

tests\fixtures\gate2_wrong_pack\manifest.json

tests\fixtures\gate2_wrong_pack\cameras\presets\bad.json
```

There is also:

``` text
tests\__pycache__\test_transforms.cpython-314.pyc
```

------------------------------------------------------------------------

# 12. M0 FILE INVENTORY

The M0 directory contains:

``` text
m0\REFERENCE_RIG.md

m0\behavior_packs\clver_m0\manifest.json

m0\behavior_packs\clver_m0\cameras\presets\m0_probe.json

m0\behavior_packs\clver_m0\functions\stop.mcfunction

m0\behavior_packs\clver_m0\functions\t00_preflight.mcfunction

m0\behavior_packs\clver_m0\functions\t01_axis_map.mcfunction

m0\behavior_packs\clver_m0\functions\t02_yaw.mcfunction

m0\behavior_packs\clver_m0\functions\t03b_pitch_cmd_neg.mcfunction

m0\behavior_packs\clver_m0\functions\t03c_pitch_script.mcfunction

m0\behavior_packs\clver_m0\functions\t03_pitch_sign.mcfunction

m0\behavior_packs\clver_m0\functions\t04b_fov_axis.mcfunction

m0\behavior_packs\clver_m0\functions\t04_fov_range.mcfunction

m0\behavior_packs\clver_m0\functions\t05_roll.mcfunction

m0\behavior_packs\clver_m0\functions\t06_spline_cap.mcfunction

m0\behavior_packs\clver_m0\functions\t07_tick_interp.mcfunction

m0\behavior_packs\clver_m0\functions\t08_key_spacing.mcfunction

m0\behavior_packs\clver_m0\functions\t09_settle.mcfunction

m0\behavior_packs\clver_m0\functions\t10_beta_off.mcfunction

m0\behavior_packs\clver_m0\scripts\main.js
```

------------------------------------------------------------------------

# 13. M0 TESTS DISCOVERED

Inspection of the M0 files revealed these test functions:

``` text
t00Preflight

t01AxisMap

t02Yaw

t03PitchSign

t04FovRange

t04bFovAxis

t05Roll

t06SplineCap

t07TickInterp

t08KeySpacing

t09Settle

t10BetaProbe
```

Corresponding function commands include:

``` mcfunction
/function t00_preflight

/function t01_axis_map

/function t02_yaw

/function t03_pitch_sign

/function t03b_pitch_cmd_neg

/function t03c_pitch_script

/function t04_fov_range

/function t04b_fov_axis

/function t05_roll

/function t06_spline_cap

/function t07_tick_interp

/function t08_key_spacing

/function t09_settle

/function t10_beta_off

/function stop
```

The source contains the startup message:

``` text
harness loaded. Run /function t00_preflight first.
```

The M0 reference document also describes a reference rig and
screenshot-based measurement workflow.

------------------------------------------------------------------------

# 14. M0 REFERENCE RIG

`m0\REFERENCE_RIG.md` states that the reference should be built once in
a flat Creative world before testing.

The documented setup includes:

-   a flat world,
-   Creative mode,
-   Peaceful,
-   a chosen origin,
-   reference markers,
-   directional references,
-   and screenshot checkpoints.

The exact procedure should be read directly from:

``` text
m0\REFERENCE_RIG.md
```

Claude should inspect the file rather than relying on this summary.

------------------------------------------------------------------------

# 15. M0 DEPLOYMENT

The M0 pack was identified as:

``` text
m0\behavior_packs\clver_m0
```

It is a Behavior Pack.

There is no separate M0 Resource Pack in the observed directory
structure.

This was not itself an error.

The immediate purpose of M0 is engine measurement.

------------------------------------------------------------------------

# 16. M0 IN-GAME ERROR --- PRIMARY BLOCKER

After implementing/deploying the M0 pack into Minecraft Bedrock, the
game produced the following error:

``` text
[Scripting][error]-[Clver M0 Measurement Pack] ReferenceError: Native function [World::sendMessage] cannot be used in early execution. at say (main.js:35)
at <anonymous> (main.js:575)
```

A second related log showed:

``` text
[Scripting][error]-Plugin [Clver M0 Measurement Pack - 1.0.0] - [main.js] ran with error:
[ReferenceError: Native function [World::sendMessage] cannot be used in early execution.
at say (main.js:35)
at <anonymous> (main.js:575)
]
```

This is the most important current M0 blocker.

------------------------------------------------------------------------

# 17. EXACT ERROR LOCATIONS

The game identifies:

``` text
main.js:35
```

and:

``` text
main.js:575
```

The reported call relationship is:

``` text
main.js:575
    ↓
say(...)
    ↓
main.js:35
    ↓
World::sendMessage
    ↓
early execution error
```

This is an observed stack trace.

The report does **not** dictate how Claude should resolve it.

Claude must inspect the actual current `main.js`, determine why the call
occurs during the relevant execution phase, and use its own sources and
engineering approach.

------------------------------------------------------------------------

# 18. M0 IMPACT OF THE ERROR

Because the script throws during startup, the intended M0 harness
initialization did not complete normally.

Therefore, at the time of this report:

``` text
M0 startup = BLOCKED
```

and the following should **not** yet be treated as successfully
measured:

``` text
T00
T01
T02
T03
T04
T04b
T05
T06
T07
T08
T09
T10
```

The existence of the function files does not prove the tests have
executed successfully in-game.

------------------------------------------------------------------------

# 19. WHAT HAS NOT YET BEEN MEASURED

No trustworthy M0 engine measurement has been established yet from the
observed session.

In particular, do not claim measured values for:

``` text
axis mapping
yaw zero
yaw sign
pitch sign
FOV range
FOV axis
roll behavior
spline cap
tick interpolation
key spacing
settling behavior
Beta API behavior
```

until the corresponding in-game test has actually executed and produced
evidence.

------------------------------------------------------------------------

# 20. M0 TESTING DIFFICULTIES OBSERVED

## Difficulty A --- Script startup error

The harness currently throws before normal measurement can begin.

Observed error:

``` text
World::sendMessage cannot be used in early execution
```

------------------------------------------------------------------------

## Difficulty B --- CLI help behavior

The validator does not treat:

``` text
--help
```

as a normal help argument.

Observed:

``` text
Not a directory: --help
```

------------------------------------------------------------------------

## Difficulty C --- Gate 1 negative test still needs an actual run

The fixture exists:

``` text
tests\fixtures\gate1_hallucinated_method.js
```

but the actual negative execution was not captured.

------------------------------------------------------------------------

## Difficulty D --- M0 requires physical in-game verification

The automated Python/TypeScript tests do not replace Minecraft
measurements.

The M0 results depend on running the Behavior Pack in the actual Bedrock
environment.

------------------------------------------------------------------------

## Difficulty E --- Visual evidence is required

Several M0 measurements are intended to be observable through the
camera/reference rig.

Therefore, successful API execution alone should not be considered
sufficient evidence for visual camera behavior.

------------------------------------------------------------------------

# 21. IMPORTANT DISTINCTION: AUTOMATED TESTS VS M0

The automated tests already provide strong evidence for the local
transform/math layer:

``` text
8/8 normal Gate 3 tests passed.
```

The deliberate sign-flip fixture also demonstrates that the tests detect
a known transform error:

``` text
6/8 passed
2 deliberate property failures
```

However, this does **not** establish that Minecraft's actual camera
coordinate conventions match the assumptions used by the implementation.

That is the purpose of M0.

Keep these two categories separate:

``` text
LOCAL MATHEMATICAL VERIFICATION
```

versus:

``` text
REAL MINECRAFT ENGINE MEASUREMENT
```

------------------------------------------------------------------------

# 22. CURRENT ACCEPTANCE STATUS

## Gate 1

``` text
Positive: PASS
Negative: NOT YET VERIFIED
```

## Gate 2

``` text
Positive: PASS
Negative: PASS / correctly rejected
```

## Gate 3

``` text
Positive: PASS / 8 of 8
Negative: PASS / correctly detected sign flip
```

## M0

``` text
Startup: BLOCKED BY SCRIPT ERROR
Measurements: NOT YET VERIFIED
```

------------------------------------------------------------------------

# 23. REQUIRED OUTCOME

The desired end state is:

``` text
Gate 1 positive test       PASS
Gate 1 negative test       correctly FAIL

Gate 2 positive test       PASS
Gate 2 negative test       correctly FAIL

Gate 3 positive test       PASS
Gate 3 negative test       correctly FAIL

M0 harness                starts without the observed startup exception

T00                        executes successfully

T01-T10                    become executable and measurable

M0 results                 recorded from actual Minecraft behavior

Evidence                    attached to the corresponding measurements
```

------------------------------------------------------------------------

# 24. DO NOT FABRICATE RESULTS

Until the actual M0 tests have been run:

Do not claim:

``` text
yaw is X
pitch is Y
roll works as Z
FOV is X
spline cap is X
interpolation is X
settle time is X
```

unless those values are supported by actual M0 evidence.

Unknown is acceptable.

An unresolved question should remain unresolved until verified.

------------------------------------------------------------------------

# 25. DO NOT CHANGE THE TEST'S INTENT

When fixing the M0 harness, preserve the intended test questions and
evidence requirements.

Do not make a test "pass" by:

-   deleting it,
-   disabling it,
-   bypassing the API call,
-   hard-coding a result,
-   suppressing the error,
-   changing expected values without evidence,
-   replacing an engine test with a local simulation,
-   or removing the measurement requirement.

The goal is to make the existing test harness actually perform its
intended measurement.

------------------------------------------------------------------------

# 26. CLAUDE'S ROLE

Claude should treat this document as a **bug report / test log /
evidence handoff**.

Claude is responsible for:

1.  Inspecting the repository.
2.  Reproducing problems where appropriate.
3.  Consulting current authoritative sources.
4.  Determining root causes independently.
5.  Choosing the correct implementation approach independently.
6.  Making the necessary changes.
7.  Running the relevant verification again.
8.  Reporting the resulting evidence.

This document intentionally does **not** prescribe implementation
details.

------------------------------------------------------------------------

# 27. NO CODE-PRESCRIPTION RULE

**Do not interpret this report as instructions to add specific code.**

The report may mention:

-   filenames,
-   line numbers,
-   API names,
-   commands,
-   stack traces,
-   observed architecture,
-   or possible conceptual causes.

These are evidence, not implementation instructions.

Claude should not blindly copy an implementation from this document.

Use current project source and authoritative documentation to decide the
correct fix.

------------------------------------------------------------------------

# 28. FINAL HANDOFF SUMMARY

### Working

``` text
npm run verify
    PASS

Gate 2 valid M0 pack
    PASS

Gate 2 invalid fixture
    correctly rejected

Gate 3 transform tests
    8/8 PASS

Gate 3 seeded sign-flip fixture
    correctly detected
```

### Not yet complete

``` text
Gate 1 negative fixture
    not yet executed

M0 startup
    blocked by World::sendMessage early-execution error

M0 measurements
    not yet established
```

### Primary observed Minecraft error

``` text
ReferenceError:
Native function [World::sendMessage] cannot be used in early execution.

at say (main.js:35)
at <anonymous> (main.js:575)
```

### Primary task

``` text
Investigate and resolve the M0 startup failure,
then re-run the actual M0 measurement process.
```

### Critical constraint

``` text
Do not guess engine behavior.
Do not fabricate M0 results.
Do not treat local math tests as substitutes for Minecraft measurements.
```

------------------------------------------------------------------------

# 29. EVIDENCE LOG

This section is intentionally concise for future updates.

## Automated evidence

``` text
2026-08-14

npm run verify
PASS

python tools\validate_pack.py m0\behavior_packs\clver_m0
PASS - no problems.

python tools\validate_pack.py tests\fixtures\gate2_wrong_pack
FAIL - 2 problem(s).
Expected negative-test result.

python tests\test_transforms.py
8/8 passed

python tests\fixtures\gate3_sign_flip.py
6/8 passed
2 expected property failures caused by seeded sign flip
```

## Minecraft evidence

``` text
2026-08-14

M0 Behavior Pack deployed.

Startup produced:

ReferenceError:
Native function [World::sendMessage] cannot be used in early execution.

at say (main.js:35)
at <anonymous> (main.js:575)
```

M0 measurement results remain pending.

------------------------------------------------------------------------

# 30. END STATE

This report should be considered a **living diagnostic log**.

When Claude fixes an issue, the next verification should be based on
actual observed output.

Do not replace this evidence with assumptions.

The next useful evidence should be:

``` text
1. Result of Gate 1 negative fixture.
2. Result of M0 startup after investigation/fix.
3. Result of /function t00_preflight.
4. Subsequent M0 test outputs and screenshots.
```

Everything beyond that should be based on actual observed results.
