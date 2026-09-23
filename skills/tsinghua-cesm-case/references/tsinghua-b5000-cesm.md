# Tsinghua B5000 CESM/CTSM Reference

## Local Configuration

This public reference intentionally contains no real username, login address, personal name, or private filesystem path. Before acting on a cluster, obtain the user's approved local values corresponding to `../config.env.example`.

```text
Login: ssh ${CLUSTER_USER}@${LOGIN_HOST}
Home: ${USER_HOME}
Shared research/data: ${GROUP_SHARE_ROOT}
CESM root: ${CESM_ROOT}
Default source: ${CESM_SRC}
Cases: ${CASE_ROOT}
Scratch/run/build: ${SCRATCH_ROOT}
Archive: ${ARCHIVE_ROOT}
Input data: ${DIN_LOC_ROOT}
Jupyter script: ${JUPYTER_SCRIPT}
```

Do not guess missing values. Verify that resolved paths remain within the intended user or project directories before copying, moving, or deleting files.

The B5000 CIME machine configuration commonly supplies values such as:

```text
CIME_OUTPUT_ROOT=${SCRATCH_ROOT}
DIN_LOC_ROOT=${DIN_LOC_ROOT}
DIN_LOC_ROOT_CLMFORC=${DIN_LOC_ROOT}/atm/datm7/
DOUT_S_ROOT=${ARCHIVE_ROOT}/$CASE
BATCH_SYSTEM=slurm
queue=all
```

## Default Create Command

```bash
cd "${CESM_SRC}/cime/scripts"
./create_newcase \
  --case "${CASE_ROOT}/CASE_NAME" \
  --res f19_g17 \
  --compset I2000Clm50BgcCropCrujra \
  --mach B5000 \
  --run-unsupported
cd "${CASE_ROOT}/CASE_NAME"
./case.setup
./case.build --skip-provenance-check
./case.submit
```

Variable fields include `CASE_NAME`, `--res`, and `--compset`. Keep `--mach B5000 --run-unsupported` fixed only when confirmed for the target installation. Before creating a case, verify that the destination does not already exist.

## XML Timing Controls

Use `env_run.xml` through `xmlchange`:

```bash
./xmlchange RUN_STARTDATE=2011-01-01
./xmlchange STOP_OPTION=nyears
./xmlchange STOP_N=13
./xmlchange CONTINUE_RUN=FALSE
./xmlchange RESUBMIT=0
```

Run length uses `STOP_OPTION` plus `STOP_N`; common options are `ndays`, `nmonths`, `nyears`, and `nsteps`. For restart or continuation cases also inspect `RUN_TYPE`, `RUN_REFCASE`, `RUN_REFDATE`, `RUN_REFTOD`, `REST_OPTION`, and `REST_N`.

## CLM Namelist Template

Write a site-appropriate version to the case-local `user_nl_clm` when hourly CLM output is requested:

```fortran
finidat = '${DIN_LOC_ROOT}/lnd/clm2/initdata_map/<initial-condition-file>.nc'
use_init_interp = .true.
hist_empty_htapes = .true.
hist_fincl1 = 'SOILICE', 'SOILLIQ', 'TG', 'TSOI', 'TWS', 'TSA', 'NEE', 'GPP', 'ALT', 'SNOW_DEPTH', 'HR', 'AR', 'TOTSOMC', 'T_SCALAR', 'W_SCALAR'
hist_nhtfrq = -1
hist_mfilt = 24
hist_avgflag_pertape = 'A'
```

Replace the initial-condition placeholder with a verified file compatible with the selected grid and compset. `hist_nhtfrq=-1` means hourly output; `hist_mfilt=24` means 24 samples per history file. Use `hist_nhtfrq=-24` for daily output when appropriate.

After editing:

```bash
./preview_namelists
grep -i "finidat\|use_init_interp\|hist_empty\|hist_fincl\|hist_nhtfrq\|hist_mfilt\|hist_avgflag\|use_" CaseDocs/lnd_in
```

`grep -i use_ CaseDocs/lnd_in` checks generated case configuration; it is not a complete source-code audit.

## DATM CruJRA Template

Write this to case-local `user_nl_datm` when replacing default forcing with CruJRA, after confirming the available year range:

```fortran
streams = 'datm.streams.txt.CruJRA.Solar 2000 2000 2023',
'datm.streams.txt.CruJRA.Precip 2000 2000 2023',
'datm.streams.txt.CruJRA.TPQW 2000 2000 2023'
```

Keep three stream files in the case directory:

```text
datm.streams.txt.CruJRA.Solar
datm.streams.txt.CruJRA.Precip
datm.streams.txt.CruJRA.TPQW
```

Use configured input-data paths rather than embedding site-private locations:

```text
Domain: ${DIN_LOC_ROOT}/atm/datm7/<crujra-domain-file>.nc
Forcing directory: ${DIN_LOC_ROOT}/atm/datm7/<crujra-forcing-directory>
```

Representative file patterns are:

```text
clmforc.CRUJRAv2.5_0.5x0.5.Prec.<year>.nc
clmforc.CRUJRAv2.5_0.5x0.5.Solr.<year>.nc
clmforc.CRUJRAv2.5_0.5x0.5.TPQWL.<year>.nc
```

Typical NetCDF forcing variables are `PRECTmms`, `FSDS`, `TBOT`, `PSRF`, `QBOT`, `WIND`, and `FLDS`. Verify them with `ncdump -h` for the actual files. DATM `variableNames` normally maps forcing variables to DATM aliases:

```xml
PRECTmms precn
FSDS swdn
TBOT tbot
WIND wind
QBOT shum
PSRF pbot
FLDS lwdn
```

After editing:

```bash
./preview_namelists
grep -A10 -i "streams" CaseDocs/datm_in
./xmlquery RUNDIR
cp datm.streams.txt.CruJRA.* "$(./xmlquery RUNDIR --value)/"
ls -lh "$(./xmlquery RUNDIR --value)"/datm.streams.txt.CruJRA.*
```

Do not let `RUN_STARTDATE` plus run length exceed the forcing coverage unless deliberate cycling is scientifically justified and explicitly requested.

## Build, Submit, and Status

Normal build and submit:

```bash
./case.build --skip-provenance-check
./case.submit
```

Useful inspections:

```bash
tail -80 CaseStatus
./xmlquery CASE CASEROOT COMPSET GRID MACH CIME_OUTPUT_ROOT RUNDIR DOUT_S_ROOT DIN_LOC_ROOT RUN_STARTDATE STOP_OPTION STOP_N CONTINUE_RUN RESUBMIT
squeue -u "${CLUSTER_USER}"
find SourceMods -type f | sort
```

If files under `SourceMods/src.clm/` are added or changed, perform a clean rebuild when required before submission.

## Jupyter on Supercomputer

Use only configured values:

```bash
ssh -L "${LOCAL_PORT}:${COMPUTE_HOST}:${REMOTE_PORT}" "${CLUSTER_USER}@${LOGIN_HOST}"
cd "${USER_HOME}"
sbatch --nodelist="${COMPUTE_HOST}" "${JUPYTER_SCRIPT}"
```

Confirm the allocated node and the Jupyter-reported URL and token rather than assuming that a requested port is available.

## Case Management Summary Fields

When asked to manage or summarize cases, report `CASE`, `CASEROOT`, `COMPSET`, `GRID`, run timing, DATM streams, CLM initialization and history settings, `RUNDIR`, `DOUT_S_ROOT`, build/run/archive evidence from `CaseStatus`, and any `SourceMods` files.
