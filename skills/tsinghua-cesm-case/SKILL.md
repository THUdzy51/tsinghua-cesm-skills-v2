---
name: tsinghua-cesm-case
description: Create, configure, validate, build, submit, inspect, and manage CESM/CTSM land-model cases on a configurable Tsinghua B5000 environment. Use when the user asks to create cases, modify CIME XML settings, configure CLM or DATM namelists, manage forcing streams and SourceMods, inspect outputs, or run Jupyter on the cluster.
---

# Tsinghua CESM Case

## Purpose

Use this skill to operate a configured CESM/CTSM workflow on the Tsinghua B5000 supercomputer. Read local configuration before using any cluster address or filesystem path; never infer private infrastructure values from this public skill.

For configurable paths, templates, stream-file examples, and check commands, read `references/tsinghua-b5000-cesm.md` when the task involves creating, modifying, checking, building, or submitting a case. Use `config.env.example` only as a placeholder schema.

## Core Rules

- Connect with `ssh "${CLUSTER_USER}@${LOGIN_HOST}"` only after the user has supplied or approved the local configuration. Request passwords only through the interactive SSH prompt and never store them.
- For Jupyter tunneling, use `ssh -L "${LOCAL_PORT}:${COMPUTE_HOST}:${REMOTE_PORT}" "${CLUSTER_USER}@${LOGIN_HOST}"`, then submit the configured Jupyter batch script if requested.
- Use `${CESM_SRC}` as the configured default CESM source tree.
- Create cases under `${CASE_ROOT}` unless the user explicitly overrides it.
- Use `--mach B5000 --run-unsupported` for `create_newcase`.
- Treat `${SCRATCH_ROOT}` as the run/build root and `${ARCHIVE_ROOT}` as the short-term archive root.
- Prefer `xmlchange` for XML settings. Do not hand-edit `env_*.xml` unless there is no reasonable CIME command.
- Put CLM changes in `user_nl_clm` and DATM changes in `user_nl_datm`. Do not edit generated `CaseDocs/*_in` as source of truth.
- After changing `user_nl_clm`, `user_nl_datm`, or XML run settings, run `./preview_namelists` and inspect `CaseDocs/lnd_in` and `CaseDocs/datm_in` before build/submit.
- For custom DATM stream files, keep copies in the case directory and copy them into `$(./xmlquery RUNDIR --value)/` before submitting.
- Use `./case.build --skip-provenance-check` for normal builds. If SourceMods or build-time changes require a clean rebuild, use the CIME clean/build workflow deliberately.

## Standard New-Case Workflow

From the supercomputer:

```bash
cd "${CESM_SRC}/cime/scripts"
./create_newcase \
  --case "${CASE_ROOT}/CASE_NAME" \
  --res RESOLUTION \
  --compset COMPSET \
  --mach B5000 \
  --run-unsupported
cd "${CASE_ROOT}/CASE_NAME"
./case.setup
```

Default user choices when unspecified:

```text
COMPSET = I2000Clm50BgcCropCrujra
RESOLUTION = f19_g17
CASE_ROOT = ${CASE_ROOT}/CASE_NAME
```

Before creating, check whether the target case directory already exists. If it exists, do not overwrite it without explicit user approval.

## Configuration Workflow

Use this order for new or modified experiments:

1. Set run timing with `xmlchange`, usually `RUN_STARTDATE`, `STOP_OPTION`, `STOP_N`, `CONTINUE_RUN`, and optionally `RESUBMIT`.
2. Edit or append case-local `user_nl_clm` for CLM initial conditions, switches, and history output.
3. Edit or append case-local `user_nl_datm` for DATM forcing streams.
4. Place custom `datm.streams.txt.*` files in the case directory.
5. Run `./preview_namelists`.
6. Inspect `CaseDocs/lnd_in`, `CaseDocs/datm_in`, and generated stream files.
7. Query `RUNDIR` and copy custom stream files there.
8. Run `./check_input_data` if input availability is uncertain.
9. Build with `./case.build --skip-provenance-check`.
10. Submit with `./case.submit` only after the user wants the job queued.

## Validation Commands

Prefer these checks before build or submit:

```bash
./xmlquery CASE CASEROOT COMPSET GRID MACH CIME_OUTPUT_ROOT RUNDIR DOUT_S_ROOT DIN_LOC_ROOT RUN_STARTDATE STOP_OPTION STOP_N CONTINUE_RUN RESUBMIT
./preview_namelists
grep -i "finidat\|use_init_interp\|hist_empty\|hist_fincl\|hist_nhtfrq\|hist_mfilt\|hist_avgflag\|use_" CaseDocs/lnd_in
grep -A10 -i "streams" CaseDocs/datm_in
ls -lh datm.streams.txt.* user_datm.streams.txt.* 2>/dev/null
ls -lh $(./xmlquery RUNDIR --value)/datm.streams.txt.* 2>/dev/null
```

For CruJRA forcing, verify representative files and variables with `ncdump -h` if the stream files or forcing years changed.

## Common User Intents

Map terse user requests as follows:

- “新建一个默认算例”: create a case under `${CASE_ROOT}`, defaulting to `I2000Clm50BgcCropCrujra`, `f19_g17`, `B5000`.
- “改开始时间/模拟长度”: use `xmlchange RUN_STARTDATE=... STOP_OPTION=... STOP_N=...`.
- “改成 CruJRA”: configure three DATM streams for Solar, Precip, and TPQW, then copy stream files into RUNDIR.
- “加输出变量”: update `hist_fincl*`, `hist_empty_htapes`, `hist_nhtfrq`, `hist_mfilt`, and `hist_avgflag_pertape` in `user_nl_clm`.
- “看模块开没开”: inspect `CaseDocs/lnd_in` with `grep -i use_` after `preview_namelists`.
- “替换 mods”: put modified source files under the case's `SourceMods/src.clm/`, then clean/rebuild as needed.
- “统一管理一系列算例”: summarize case roots, XML timing, DATM streams, CLM history fields, build status, run status, scratch path, and archive path.

## Reporting Style

When reporting back, use concise academic Chinese. State what was created or changed, list exact paths and commands, and distinguish completed actions from recommended checks. For modeling choices, explain scientific implications briefly, especially for forcing data, initial conditions, spinup, output frequency, and process switches.
