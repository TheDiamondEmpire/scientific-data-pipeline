# Scientific Data Pipeline: Research Report

> **Where the code is.** The experiment scripts that produced these results are part of the source package, which is distributed separately (see [SOURCE-ACCESS.md](../SOURCE-ACCESS.md)). The raw results they wrote are included in this repository under [`results/`](../results/).

<p align="center"><img src="assets/banner.svg" alt="Scientific Data Pipeline" width="100%"></p>

> **Status: protocol v0.2, results recorded.** The hypotheses, methods and statistical plan were written in v0.1 before anything ran; v0.2 adds explicit decision rules (also fixed before the final run) and the measured results. All numbers below come from files in [`results/`](../results/tables.md) written by scripts in this repository, from one uninterrupted run. Faults in E1 and E2 are SYNTHETIC corruptions of real data; the format study also uses a SYNTHETIC gridded field. Three of the four hypotheses (H1, H3, H4) were not supported as stated; H2 meets its pre-stated rule but with a small effect.

## Abstract

This study asks how effectively automated validation rules detect injected data faults in scientific time series, and what the storage, read-time and provenance costs are. A local pipeline (own DAG orchestrator, Parquet and Zarr outputs, lineage, versioned releases) was built and evaluated on a small REAL dataset (eight NOAA GHCN-Daily stations, 2015 to 2019, 43,134 daily observations) with SYNTHETIC faults injected under fixed seeds. Findings: range, gap and duplicate faults were detected completely, but those are detectable by construction; unit faults were detected at 0.846 recall (95% CI 0.843 to 0.850), well below the pre-stated 0.95, because precipitation unit errors (mm written as inches) were never detected; slow drift was mostly missed (0.140 recall at a 10 degC ramp) and the statistical rule that helped a little flagged 0.24% of the unmodified real rows. Parquet and Zarr cut storage 13.7 to 23.2 times versus CSV and read time more than 10 times when warm, but not when cold, so H3 as stated is not supported. Capturing lineage added a median 30% to 52% to the wall-clock runtime of this small pipeline, so H4 (under 5%) is not supported. Ten clean runs in separate processes, with different hash seeds, thread caps and directories, produced identical hashes for every deterministic artefact (10 of 10; Wilson 95% interval 0.72 to 1.00).

## 1. Research question

How effectively do automated validation rules detect realistic data faults in scientific time-series and gridded data, and what are the throughput, storage and provenance costs of doing so?

Scope actually covered: injected faults in time-series data (real station data); the gridded case is covered only for the storage and read-time comparison, on a generated field. Throughput in GB per hour was not measured (see Deviations).

## 2. Hypotheses and decision rules

Decision rules were added in v0.2 before the final run and were not changed afterwards.

- **H1:** Rule-based validation detects at least 95% of injected unit, range and gap faults with a false-positive rate under 1%. *Rule:* supported only if the 95% CI lower bound of recall is at least 0.95 for unit, range and gap faults (full rule set) and the flag rate on the unmodified base is under 1%.
- **H2:** Statistical outlier rules add recall for subtle drift faults but raise false positives enough to require human review. *Rule:* supported if the drift recall gain of schema+range+statistical over schema+range has a 95% CI lower bound above 0 and the statistical rule flags at least 0.1% of the clean base rows.
- **H3:** Parquet and Zarr outputs cut storage by at least 5x and read time by at least 10x versus CSV for analytical queries. *Rule:* supported for a dataset only if the CSV-to-format byte ratio is at least 5 and the 95% CI lower bound of the CSV-to-format median-latency ratio is at least 10 for every query, cold and warm, for both Parquet and Zarr.
- **H4:** Capturing full provenance adds under 5% to pipeline runtime. *Rule:* supported only if the 95% CI upper bound of the median wall-clock overhead is below 5% at every data volume tested.

## 3. Background

Positioning references (not a systematic review). Only the NOAA items were checked against their original record during this work (the NCEI metadata record C00861); the others are cited from memory and must be verified before publication.

- GHCN-Daily and its quality assurance: Menne et al. (2012), overview article, and the dataset DOI (both checked against the NCEI record). Durre et al. (2010) describe the automated quality assurance of daily surface observations whose flags appear in the data used here.
- FAIR principles: Wilkinson et al. (2016).
- Declarative data-quality checks at scale: Schelter et al. (2018), the approach behind tools such as Deequ and Great Expectations; this project uses a much smaller, own rule engine.
- Provenance models: the W3C PROV data model (Moreau and Missier, 2013); the lineage here is PROV-inspired, not PROV-O.
- Metadata conventions: CF Conventions (version 1.8) and the DataCite Metadata Schema 4.x; the checks in this repository cover only a minimal subset.
- Array and columnar formats: the Zarr specification (v3) and Apache Parquet.

## 4. Method

### 4.1 System under test

The pipeline in this repository (see the [README](../README.md)): ingest into a content-addressed raw zone, parse, harmonize units, validate (schema, range, statistical, gap checks), quarantine, curate to hive-partitioned Parquet and a Zarr v3 store with CF attributes, then build a versioned release. A local DAG orchestrator provides retries, resumable stages and lineage capture. Object storage is the local filesystem.

### 4.2 Environment and datasets

Hardware: AMD64 (Family 25, Model 1) processor, 4 physical and 8 logical cores, 30.0 GB RAM, Windows 11. Python 3.12.13; numpy 2.5.4, pandas 3.1.0, pyarrow 26.0.0, duckdb 2.0.0, zarr 3.4.1, xarray 2026.9.0, scipy 1.18.1. All experiments were pinned to two logical cores (indices 0 and 1, the least loaded at the start). The machine was shared with other builds and applications: mean CPU load across all cores was 100% before and 97% after the run (`results/run_meta.json`). Timings are therefore noisy and are reported with that caveat.

**Real dataset (the only observed data).** NOAA NCEI GHCN-Daily, version 3.35 (readme), retrieved 2026-10-23 from `https://www.ncei.noaa.gov/pub/data/ghcn/daily/` (landing page `https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc:C00861`, dataset DOI 10.7289/V5D21VHZ). Extract: eight stations (six U.S. first-order airport stations and two cooperative-observer stations with natural gaps), years 2015 to 2019, elements TMAX, TMIN, PRCP; 43,134 observations, of which four carry a NOAA quality flag. Terms as stated in the NCEI record: free electronic download, citation requested (Menne et al. 2012), no warranty of accuracy; the record states no redistribution restriction and the 0.4 MB extract is checked in with attribution (this is not legal advice). Readme SHA-256 `0e4e6f0e6947cf1add884c328a01760f31c56ef5fb86b85ed57dd5dc4c4aeaa6`. Hash of the whole inbox directory (all files below plus `SOURCE.json`): `439380ad618a415d3c21e2364b8f2e879666320da96d1ef05763d012db069d6a`.

| File | Upstream bytes | Upstream SHA-256 | Extract SHA-256 |
|---|---|---|---|
| ghcnd-stations.txt (8 lines kept) | 11,441,354 | `428eabc955883187935bce283766d5646e3ed7484b92eb0803874c0e8dd9b342` | `a3ae47b7a2adf7dbbc595f271674e8ff17b112d35b5105bf9513904950c6c36f` |
| USC00040693.dly | 2,184,300 | `0ae9be36680993cf72c60d7f03e999fe3fea0636fd2ae308d3808beb0ce87083` | `ab113d28fbe7e6fe1b8fe1f9c4df4bc5de7ea75277177b900c540df3d03013f2` |
| USC00051564.dly | 2,439,450 | `80ba6ab284456f56676b397f25c6969d0004d00d588cc5e7fe6d4293630eb1fb` | `750dc97ae63a49619c75f72d353e507ecca62c90239b9395c71927bddab24792` |
| USW00003017.dly | 2,199,420 | `4df8046799e62e36cca6b9fc5511e5cf7c3191bd9f54d635fef821e957e6b9cc` | `5330a3d4b4290ba66b3fcfaed5e40c0ce29956cb549e04b1699b46b0ad85bfc8` |
| USW00012839.dly | 4,317,030 | `7a8c50dc2cdf3c83603e77511be3937b28e3bcd7a656ce350dc2980f1f53ce2b` | `0b4dc7fcf63fe91c73f3fc911a5d749a58adf4e2c9bb13e100b56344fc739d49` |
| USW00014922.dly | 5,087,880 | `e134d5402043c97fbe0a7d1fe4d128bc2813da4287b91a99df6d26a79a204347` | `9be3f319ff310c9ea91bd38520b18b23d379d5ce86dfdd93003473f2da886d59` |
| USW00023174.dly | 4,105,350 | `4bfcaddc3a8f6c075fde709974c404de77ed61f2a81ee8309762835e1c128128` | `6ddb4c2727390367d50628b9a2d028712529de62f3d6395afc21a85a397b707d` |
| USW00024233.dly | 4,409,640 | `4e739e29d9aa4aa27db42743c985fda831e9efdbc657189592af661b3962bd1e` | `61ca4a8cdefef9146c62ad507d18fb2e8b1e8eae31da851a8f53d45d6aa2c173` |
| USW00094728.dly | 4,806,810 | `9a1e4af2a8af07e96297fd97f7f5702930aea1bd4bec4a916af83cada20719a7` | `61e58adcbb5ea0051ed2cabbf6f1cfcf861c6a01e1134a325cc0ccbf2cd389d4` |

Upstream files change daily; re-downloading later may give different bytes. The checked-in extract (`tests/fixtures/ghcn`) is what the experiments ran on (`sdp fetch --offline` restores it).

**Clean base for fault injection.** The 43,130 real rows whose NOAA quality flag is blank (the four flagged rows are excluded, and the source-flag rule is switched off in E1 and E2).

**Synthetic dataset (clearly not observed).** For E3 only: a generated 240 x 72 x 144 float32 temperature field (about 2.5 million values; latitudinal mean, annual cycle, smooth random anomalies and small noise, rounded to 0.1 K) from a numpy generator with fixed seed 20260101 (`src/sdp/synth.py`). It is smooth and rounded, so it compresses very well; ratios on it may be optimistic for real gridded products.

### 4.3 Baselines

- No validation (rule set `none`): detects nothing by definition.
- Schema-only validation (rule set `schema`).
- CSV storage: queried with DuckDB's CSV reader (not a pandas workflow; see Deviations), plus gzip-compressed CSV.

### 4.4 Independent variables

Validation rule set (E2); fault type and rate (E1), plus drift magnitude (peak 5, 10, 20 degC at 3%); storage format and chunking/partitioning as built (E3); provenance on or off (E4); data volume (E4 only: 2, 4 and 8 stations of the real extract; the whole extract is under 0.4 MB).

### 4.5 Fault types (SYNTHETIC)

Injected into the clean base under seed `sha256(salt|fault|rate|rep)`, one fault type per run, at 1, 3 and 5% of rows:

- **unit:** a value written in the wrong unit with the unit label unchanged (degC to degF for TMAX and TMIN, mm to inches for PRCP; only rows whose value changes).
- **range:** a value replaced by a physically impossible one. Out of bounds by construction.
- **gap:** runs of one to five consecutive rows of a series removed. Detectable by construction (the expected window is declared).
- **drift:** a 30-day linear ramp from 0 to plus or minus 10 degC added to TMAX or TMIN.
- **duplicate:** exact copies of random rows appended. Detectable by construction (key uniqueness).

Scoring: recall of injected units; "precision of new flags" is the share of units flagged on the injected table, and not already flagged on the unmodified base, that were injected. The flag rate on the unmodified base is an upper bound on the false-positive rate because that data is real and may contain genuine extremes or defects.

### 4.6 Metrics

Fault detection recall and precision by fault type (F1 not reported); storage bytes; cold and warm read latency and scan throughput in MB per second of decoded values (not GB per hour); provenance overhead in percent of wall-clock and of process CPU time; output hash agreement.

## 5. Experiment matrix

| ID | Experiment | Manipulation | Measured | Units of work |
|---|---|---|---|---|
| E1 | Fault detection | Inject unit, range, gap, drift and duplicate faults at 1, 3 and 5%; drift peaks 5 and 20 degC at 3%; full rule set | Recall, precision of new flags, base flag rate | 20 seeds per cell, 19 cells |
| E2 | Rule-set ablation | none, schema, schema+range, schema+range+statistical; same injections, 3% | Recall gain versus base flag-rate cost | 20 paired seeds, 5 fault types |
| E3 | Format study | CSV, gzip CSV, Parquet, Zarr on identical data (REAL station data and SYNTHETIC grid) | Bytes, cold and warm latency, scan throughput | 12 cold and 24 warm samples per cell |
| E4 | Provenance cost | Lineage capture on vs. off, interleaved pairs, 2, 4 and 8 stations | Wall-clock and CPU time overhead | 12 pairs per volume |
| E5 | Determinism | 10 clean runs in separate processes, varying hash seed, thread cap and directories, plus a perturbed-input control | Hash agreement | 10 runs plus 1 control |

Commands: `uv run python -m experiments.all` runs everything (resumable); `uv run python -m experiments.e1` to `e5` run one.

## 6. Statistical plan

- **Replication:** run counts fixed in the scripts before the final run: 20 seeds (E1, E2), 12 cold reps with 2 warm samples each (E3), 12 pairs per volume (E4), 10 runs (E5).
- **Summaries:** means with percentile-bootstrap 95% CIs (5,000 resamples, seeded) for recall and precision; medians with bootstrap CIs for timings; Wilson intervals for agreement proportions. Spread is shown by the CI; CIs over injection seeds describe variability of the injection on one fixed base dataset, not variability across datasets.
- **Comparisons:** paired Wilcoxon signed-rank tests (E2 across seeds, E4 across pairs).
- **Effect sizes:** Cliff's delta (E2) and the CI of the mean difference.
- **Multiple comparisons:** Holm adjustment over the 15 paired comparisons of E2. E1, E3 and E5 report intervals without hypothesis tests; E4 reports one unadjusted Wilcoxon p per cell.
- **No silent exclusions:** no outliers were removed. A warm-up pipeline run before E4 is untimed.
- **Negative results are results:** see Section 8.

## 7. Results

Full tables, including every latency cell: [`results/tables.md`](../results/tables.md); raw per-run data: `results/raw/` and `results/e*_runs.csv`, `results/e3_trials.csv`, `results/e4_pairs.csv`. Run metadata (hardware, load, hashes, single-invocation flag): [`results/run_meta.json`](../results/run_meta.json).

### E1. Fault detection (SYNTHETIC faults on REAL data)

**Manipulation:** Inject unit, range, gap, drift and duplicate faults at 1 to 5% rates (20 seeds per cell)
**Measured:** Recall and precision of new flags by fault type, full rule set

| Condition (SYNTHETIC fault, rate) | n runs | Recall (mean) | 95% CI | Notes |
|---|---|---|---|---|
| drift, 1% | 20 | 0.147 | [0.129, 0.166] | precision of new flags 0.64; about 450 injected per run, drift peak 10 degC; late-ramp recall 0.26 |
| drift, 3% | 20 | 0.024 | [0.020, 0.029] | precision of new flags 0.67; about 1320 injected per run, drift peak 5 degC; late-ramp recall 0.04 |
| drift, 3% | 20 | 0.138 | [0.125, 0.150] | precision of new flags 0.63; about 1320 injected per run, drift peak 10 degC; late-ramp recall 0.24 |
| drift, 3% | 20 | 0.374 | [0.354, 0.392] | precision of new flags 0.63; about 1320 injected per run, drift peak 20 degC; late-ramp recall 0.59 |
| drift, 5% | 20 | 0.134 | [0.126, 0.142] | precision of new flags 0.61; about 2160 injected per run, drift peak 10 degC; late-ramp recall 0.23 |
| duplicate, 1% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 1.00; about 431 injected per run |
| duplicate, 3% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 1.00; about 1294 injected per run |
| duplicate, 5% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 1.00; about 2156 injected per run |
| gap, 1% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 0.99; about 433 injected per run |
| gap, 3% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 0.99; about 1295 injected per run |
| gap, 5% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 0.99; about 2158 injected per run |
| range, 1% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 0.75; about 431 injected per run |
| range, 3% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 0.75; about 1294 injected per run |
| range, 5% | 20 | 1.000 | [1.000, 1.000] | precision of new flags 0.76; about 2156 injected per run |
| unit, 1% | 20 | 0.847 | [0.839, 0.854] | precision of new flags 0.65; about 431 injected per run; temperature rows recall 0.97; precipitation rows recall 0.00 |
| unit, 3% | 20 | 0.849 | [0.843, 0.854] | precision of new flags 0.67; about 1294 injected per run; temperature rows recall 0.96; precipitation rows recall 0.00 |
| unit, 5% | 20 | 0.844 | [0.841, 0.847] | precision of new flags 0.67; about 2156 injected per run; temperature rows recall 0.96; precipitation rows recall 0.00 |

Pooled over the three rates (default drift peak 10 degC):

| Fault type | n runs | Recall (mean) | 95% CI | H1 check (CI lower bound >= 0.95) |
|---|---|---|---|---|
| drift | 60 | 0.140 | [0.132, 0.148] | n/a (not part of H1) |
| duplicate | 60 | 1.000 | [1.000, 1.000] | n/a (not part of H1) |
| gap | 60 | 1.000 | [1.000, 1.000] | pass |
| range | 60 | 1.000 | [1.000, 1.000] | pass |
| unit | 60 | 0.846 | [0.843, 0.850] | FAIL |

Flag rate of the full rule set on the unmodified clean base (REAL data, 43130 rows): 103 rows = 0.2388%; 694 natural missing station-element-days are reported as gaps. Flags by rule: range.gap=694, statistical.climatology_outlier=103.

Real NOAA-QFLAG rows (descriptive, n=4): 1 of 4 were also flagged by the schema+range+statistical rules (source-QFLAG rule off); our rules flagged 104 rows in total on the full real data.

### E2. Rule-set ablation (SYNTHETIC faults on REAL data)

**Manipulation:** none vs. schema vs. schema+range vs. schema+range+statistical (3% faults, 20 paired seeds)
**Measured:** Recall gain versus false-positive cost

| Rule set | n runs | Macro recall over 5 fault types (mean) | 95% CI | Base flag rate (REAL clean rows) |
|---|---|---|---|---|
| none | 20 | 0.000 | [0.000, 0.000] | 0.0000% |
| schema | 20 | 0.200 | [0.200, 0.200] | 0.0000% |
| schema+range | 20 | 0.766 | [0.764, 0.768] | 0.0000% |
| schema+range+statistical | 20 | 0.797 | [0.795, 0.799] | 0.2388% |

Recall by fault type (mean over seeds, bootstrap 95% CI):

| Fault type | none | schema | schema+range | schema+range+statistical |
| --- | --- | --- | --- | --- |
| unit | 0.000 [0.00, 0.00] | 0.000 [0.00, 0.00] | 0.737 [0.73, 0.74] | 0.849 [0.84, 0.85] |
| range | 0.000 [0.00, 0.00] | 0.000 [0.00, 0.00] | 1.000 [1.00, 1.00] | 1.000 [1.00, 1.00] |
| gap | 0.000 [0.00, 0.00] | 0.000 [0.00, 0.00] | 1.000 [1.00, 1.00] | 1.000 [1.00, 1.00] |
| drift | 0.000 [0.00, 0.00] | 0.000 [0.00, 0.00] | 0.093 [0.08, 0.11] | 0.138 [0.13, 0.15] |
| duplicate | 0.000 [0.00, 0.00] | 1.000 [1.00, 1.00] | 1.000 [1.00, 1.00] | 1.000 [1.00, 1.00] |

Paired comparisons across seeds (Wilcoxon signed-rank; Holm adjustment over all rows):

| Fault | Comparison | n pairs | Mean recall gain | 95% CI | Cliff's delta | Holm p |
|---|---|---|---|---|---|---|
| unit | schema vs none | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| unit | schema+range vs schema | 20 | 0.737 | [0.732, 0.742] | 1.00 | 0.0011 |
| unit | schema+range+statistical vs schema+range | 20 | 0.112 | [0.108, 0.115] | 1.00 | 0.0011 |
| range | schema vs none | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| range | schema+range vs schema | 20 | 1.000 | [1.000, 1.000] | 1.00 | 0.00012 |
| range | schema+range+statistical vs schema+range | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| gap | schema vs none | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| gap | schema+range vs schema | 20 | 1.000 | [1.000, 1.000] | 1.00 | 0.00012 |
| gap | schema+range+statistical vs schema+range | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| drift | schema vs none | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| drift | schema+range vs schema | 20 | 0.093 | [0.082, 0.106] | 1.00 | 0.0011 |
| drift | schema+range+statistical vs schema+range | 20 | 0.044 | [0.038, 0.051] | 0.71 | 0.0011 |
| duplicate | schema vs none | 20 | 1.000 | [1.000, 1.000] | 1.00 | 0.00012 |
| duplicate | schema+range vs schema | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |
| duplicate | schema+range+statistical vs schema+range | 20 | 0.000 | [0.000, 0.000] | 0.00 | all differences zero |

Base flag rate added by the statistical rule: 0.2388%. H2 checks: {'drift_gain_ci_low_gt_0': True, 'stat_flag_rate_ge_0.1pct': True}.

### E3. Format study (REAL station data and SYNTHETIC grid)

**Manipulation:** CSV vs. gzip CSV vs. Parquet vs. Zarr on identical data
**Measured:** Size, cold and warm read latency, scan throughput (full latency table in `results/tables.md`)

Storage (deterministic; logical file bytes):

| Dataset | Format | Bytes | Files | CSV / format |
|---|---|---|---|---|
| grid | csv | 72,368,657 | 1 | 1.00 |
| grid | csv_gz | 9,891,846 | 1 | 7.32 |
| grid | parquet | 3,124,110 | 1 | 23.16 |
| grid | zarr | 3,320,707 | 68 | 21.79 |
| real | csv | 1,404,659 | 1 | 1.00 |
| real | csv_gz | 166,246 | 1 | 8.45 |
| real | parquet | 102,718 | 1 | 13.67 |
| real | zarr | 70,812 | 31 | 19.84 |

Median latency ratio, CSV over format (higher means faster than CSV):

| Dataset | Format | Query | Kind | CSV / format | 95% CI | CI lower >= 10 |
|---|---|---|---|---|---|---|
| real | csv_gz | scan | cold | 0.9 | [0.7, 1.3] | no |
| real | csv_gz | scan | warm | 1.0 | [0.7, 1.4] | no |
| real | csv_gz | series | cold | 1.1 | [0.7, 1.6] | no |
| real | csv_gz | series | warm | 0.9 | [0.7, 1.3] | no |
| real | csv_gz | slice | cold | 0.7 | [0.5, 1.2] | no |
| real | csv_gz | slice | warm | 1.0 | [0.8, 1.2] | no |
| real | parquet | scan | cold | 1.7 | [1.2, 3.4] | no |
| real | parquet | scan | warm | 23.1 | [15.8, 32.0] | yes |
| real | parquet | series | cold | 2.7 | [1.6, 4.0] | no |
| real | parquet | series | warm | 18.7 | [14.4, 28.3] | yes |
| real | parquet | slice | cold | 2.4 | [1.6, 4.0] | no |
| real | parquet | slice | warm | 19.9 | [14.3, 25.7] | yes |
| real | zarr | scan | cold | 3.8 | [2.9, 5.3] | no |
| real | zarr | scan | warm | 98.6 | [73.6, 131.3] | yes |
| real | zarr | series | cold | 4.2 | [2.6, 5.9] | no |
| real | zarr | series | warm | 10.9 | [8.4, 15.5] | no |
| real | zarr | slice | cold | 4.4 | [3.2, 6.6] | no |
| real | zarr | slice | warm | 27.7 | [22.2, 35.1] | yes |
| grid | csv_gz | scan | cold | 0.7 | [0.6, 1.1] | no |
| grid | csv_gz | scan | warm | 0.7 | [0.6, 0.9] | no |
| grid | csv_gz | series | cold | 0.7 | [0.5, 0.9] | no |
| grid | csv_gz | series | warm | 0.6 | [0.5, 0.7] | no |
| grid | csv_gz | slice | cold | 0.9 | [0.6, 1.3] | no |
| grid | csv_gz | slice | warm | 0.8 | [0.7, 1.0] | no |
| grid | parquet | scan | cold | 7.1 | [5.0, 11.6] | no |
| grid | parquet | scan | warm | 40.9 | [33.0, 47.7] | yes |
| grid | parquet | series | cold | 8.4 | [5.1, 11.0] | no |
| grid | parquet | series | warm | 27.8 | [24.2, 36.8] | yes |
| grid | parquet | slice | cold | 19.3 | [9.6, 26.3] | no |
| grid | parquet | slice | warm | 115.7 | [87.9, 151.3] | yes |
| grid | zarr | scan | cold | 5.1 | [3.9, 7.6] | no |
| grid | zarr | scan | warm | 68.0 | [57.7, 82.4] | yes |
| grid | zarr | series | cold | 17.6 | [11.8, 24.8] | yes |
| grid | zarr | series | warm | 42.8 | [38.2, 53.2] | yes |
| grid | zarr | slice | cold | 23.8 | [16.3, 31.5] | yes |
| grid | zarr | slice | warm | 73.0 | [52.2, 89.5] | yes |

Maximum relative difference between formats' query answers: {'grid': 5.321679768515904e-10, 'real': 8.423006041536368e-15}.

Selected medians (seconds), grid, cold or warm: CSV scan 1.14 or 1.08; Parquet scan 0.160 or 0.026; Zarr scan 0.223 or 0.016. Scan throughput of decoded values on the grid, warm: CSV 9.2, Parquet 377.6, Zarr 628.6 MB/s. Every format returned identical query answers (maximum relative difference 5.3e-10 on the grid). Cold means the first query through a freshly opened handle (includes opening); the operating-system file cache was not flushed, so cold is not a disk-cold measurement.

### E4. Provenance cost (REAL data)

**Manipulation:** Lineage capture on vs. off, 12 interleaved pairs per volume
**Measured:** Runtime overhead

| Stations | Metric | n pairs | Off median s | On median s | Overhead % (median) | 95% CI | Median diff s | Wilcoxon p | CI upper < 5% |
|---|---|---|---|---|---|---|---|---|---|
| 2 | wall_s | 12 | 5.29 | 7.38 | 30.7 | [19.2, 55.6] | 1.74 | 0.0005 | no |
| 2 | cpu_s | 12 | 1.86 | 2.57 | 42.9 | [36.0, 45.9] | 0.79 | 0.0005 | no |
| 4 | wall_s | 12 | 3.36 | 5.05 | 51.6 | [18.8, 61.6] | 1.59 | 0.0005 | no |
| 4 | cpu_s | 12 | 2.34 | 3.30 | 44.5 | [36.8, 50.1] | 0.97 | 0.0005 | no |
| 8 | wall_s | 12 | 4.80 | 6.30 | 29.7 | [19.6, 45.7] | 1.34 | 0.0015 | no |
| 8 | cpu_s | 12 | 3.35 | 4.62 | 32.6 | [26.6, 41.1] | 1.13 | 0.0005 | no |

### E5. Determinism (REAL data)

**Manipulation:** 10 clean runs, each in a separate process with a different PYTHONHASHSEED, a thread cap alternating between 1 and 2, and different working and current directories
**Measured:** Hash agreement

| Artefact | n clean runs | Distinct hashes | Agreement with modal | 95% CI (Wilson) | Expected | Perturbed-input control differs |
|---|---|---|---|---|---|---|
| release_tree | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| curated_tree | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| quarantine_tree | 10 | 1 | 10/10 | [0.72, 1.00] | identical | False |
| lineage_core | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| all_deterministic_files | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| run_log_volatile | 10 | 10 | 1/10 | [0.02, 0.40] | differ (volatile control) | True |

Files compared per run: 180. Varied between runs: PYTHONHASHSEED, thread cap (1 or 2), working directory path, current directory.

## 8. Discussion

**H1 is not supported.** Range, gap and duplicate faults were detected completely (recall 1.000, CI lower bound 1.000), but these three are detectable by construction (an out-of-bounds value, a declared expected day missing, a repeated key), so they verify the plumbing rather than the rules' strength. The informative case, unit faults, reached 0.846 (CI 0.843 to 0.850), below the 0.95 bar. The split explains why: temperature unit errors were caught in 96 to 97% of rows (Fahrenheit values above 60 leave the physical range; the cooler ones were caught by the TMAX-below-TMIN rule or the statistical rule), while precipitation unit errors (millimetres written as inches) were never detected (recall 0.00), because shrunken precipitation values remain physically plausible and the statistical rule is not applied to precipitation. The flag-rate criterion on the unmodified base passed (0.24%, under 1%), but all 103 of those flags come from the statistical rule on real data and most are plausibly genuine heat events; without labels this is unverified. Precision of new flags was 0.61 to 0.76 for value faults because the TMAX-below-TMIN rule also flags the untouched partner row; gap and duplicate flags were almost exclusively injected units (0.99 and 1.00).

**H2 meets its pre-stated rule, with a small effect.** Adding the statistical rule raised drift recall from 0.093 to 0.138 (mean gain 0.044, CI 0.038 to 0.051, Holm-adjusted p 0.0011) and flagged 0.24% of clean rows (103 rows, above the 0.1% threshold). That is a real but modest gain: about 86% of drifted rows still went undetected at a 10 degC ramp, and at a 5 degC peak recall was 0.024. Drift is hard for a day-of-year robust z-score because early ramp values are within normal variability; recall rose to 0.374 at a 20 degC peak. Whether the 103 flags "require human review" cannot be judged from this data: only 1 of the 4 real NOAA-flagged rows was also flagged by the rules, and n = 4 supports no conclusion about agreement with NOAA's own quality assurance. The drift recall already present in the schema+range step (0.093) most plausibly comes from the TMAX-below-TMIN rule (the only row-level range-group rule that a 10 degC shift can trigger), not from any drift-specific rule; this attribution was not isolated by a separate experiment.

**H3 is supported for storage and not as stated for read time.** Parquet and Zarr were 13.7 and 19.8 times smaller than CSV on the real data and 23.2 and 21.8 times on the synthetic grid; even gzip CSV was 7 to 8 times smaller but no faster to read (ratios 0.6 to 1.1). Warm reads met the 10x criterion (CI lower bound at least 10) in 11 of 12 Parquet and Zarr cells (the exception: Zarr series queries on the tiny real data, ratio 10.9 with CI lower bound 8.4). Cold reads met it only for Zarr series and slice queries on the grid; Parquet cold ratios were 1.7 to 19.3 and Zarr cold ratios on the real data 3.8 to 4.4, so the pre-stated rule over all queries, cold and warm, fails. The real dataset is too small (millisecond reads) for latency to say much; the grid is synthetic and compresses unusually well. CSV was read through DuckDB, which is already fast on CSV; a pandas baseline would likely widen the gap.

**H4 is not supported.** Lineage capture added a median 29.7% to 51.6% of wall-clock time (CI upper bounds 45.7 to 61.6%) and 32.6% to 44.5% of CPU time, a median 1.3 to 1.7 seconds on pipelines of 3 to 7 seconds. The cost is fixed-size work (hashing every stage output tree, capturing the environment, writing state and lineage files) that is large relative to a pipeline that processes under half a megabyte, and the implementation was not optimized. Whether the relative overhead would fall below 5% on gigabyte-scale inputs was not tested; the data volume range here (2 to 8 stations) showed no downward trend.

**E5: determinism holds in the tested conditions.** All 10 runs produced identical hashes for the release tree, the curated tree, the quarantine tree, the lineage core and the 180 deterministic files together, despite different hash seeds, thread caps and directories. The Wilson 95% interval for the agreement rate is 0.72 to 1.00, so ten runs cannot rule out a rare disagreement. The controls show the comparison can see differences: the volatile run log differed in all 10 runs, and a single-character change to one input value changed the release, curated and lineage hashes. It did not change the quarantine tree, as expected, because the altered value was not quarantined and the quarantine holds only failing rows.

**Practical guidance.** Treat the three by-construction faults as regression checks, not as evidence of detection strength. Unit errors need unit-aware checks per variable (for example cross-variable relations or per-station climatology for precipitation); statistical rules need labelled real defects before their thresholds are trusted. Parquet or Zarr is a clear storage and warm-read win; do not expect the same factor on cold first reads. If provenance cost matters, optimize the hashing (incremental or sampled) before adopting full capture on tiny, frequently rerun pipelines.

## 9. Threats to validity

- **Injected faults are synthetic and designed by the same author as the rules.** Range, gap and duplicate faults are detectable by construction. The unit and drift magnitudes are choices; other magnitudes would give other recalls (the drift cells show this).
- **Real defects are almost absent.** Only four real rows carry a NOAA quality flag and no manually labelled defects were produced; the false-positive rate is bounded only by the flag rate on real data that may contain genuine extremes.
- **One small, U.S.-only dataset** of eight stations and five years; results are reported for this dataset only. CIs over seeds reflect injection variability on one fixed base, not sampling variability across datasets or stations.
- **Untuned thresholds.** The z threshold (4.0), climatology window (plus or minus 7 days) and bounds were fixed in advance and not tuned; results would change with other values.
- **Timing under contention.** The machine ran at 97 to 100% CPU from other work; E3 and E4 timings are noisy, the run was pinned to two cores that were themselves busy, and the order of formats was randomized to spread, not remove, the noise. Cold reads are not disk-cold.
- **Synthetic grid** is smooth and rounded; compression and read ratios may be optimistic. CSV was read with DuckDB.
- **Provenance overhead is a property of this implementation** on a tiny pipeline, not of provenance capture in general.
- **Local-only stand-ins:** the local filesystem replaces object storage; Windows file scanning can affect I/O timings. Other operating systems were not tested; CI has not run.
- **Researcher degrees of freedom:** protocol choices were fixed before the final run; every deviation and rerun is disclosed in Deviations and the change log. A two-seed pilot of every experiment was run during development to test the harness and its numbers were seen, but no parameter was changed afterwards.

## 10. Reproducibility checklist

- [ ] Code tagged at the commit used for the reported results (the folder is not a git repository; the code content hashes in `results/run_meta.json` identify the code instead)
- [ ] Environment pinned (lockfile and container image digest recorded): the lockfile hash is recorded; no container image exists, so this item is only partly true
- [x] Random seeds and run counts recorded
- [x] Raw results published (CSV and per-unit JSON) alongside the analysis scripts (no notebook is used)
- [x] Hardware or service configuration described in full (Section 4.2 and `results/run_meta.json`)
- [x] One command reproduces every table (`uv run python -m experiments.all`, which also renders `results/tables.md`; there are no figures)
- [x] Data sources, licenses and versions listed (Section 4.2)

## Deviations from the protocol

1. The "CSV storage with pandas workflow" baseline was implemented as CSV read through DuckDB, plus gzip CSV.
2. Throughput in GB per hour was not measured; scan throughput is reported in MB per second of decoded values (E3). End-to-end pipeline throughput was not measured.
3. Data volume was varied only in E4 and only from 2 to 8 stations (under 0.4 MB in total); no gigabyte-scale volume was tested.
4. Decision rules for H1 to H4 were added in v0.2 (before the final run).
5. F1 was not computed; precision is "precision of new flags" as defined in Section 4.5.
6. Holm adjustment was applied within E2 only; E4 reports unadjusted Wilcoxon p values.
7. Manually labelled real defects were not produced; the only real labels are four NOAA quality flags.
8. No Dagster, Prefect, S3 or Docker components were used; the container image digest therefore does not exist.

## 11. References

- Wilkinson, M. D. et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. Scientific Data 3:160018.
- Menne, M. J., Durre, I., Vose, R. S., Gleason, B. E., Houston, T. G. (2012). An overview of the Global Historical Climatology Network-Daily database. Journal of Atmospheric and Oceanic Technology 29, 897-910. doi:10.1175/JTECH-D-11-00103.1 (checked against the NCEI record)
- Menne, M. J., Durre, I., Korzeniewski, B., McNeill, S., Thomas, K., Yin, X., Anthony, S., Ray, R., Vose, R. S., Gleason, B. E., Houston, T. G. (2012). Global Historical Climatology Network - Daily (GHCN-Daily), Version 3. NOAA National Climatic Data Center. doi:10.7289/V5D21VHZ (checked against the NCEI record)
- Durre, I., Menne, M. J., Gleason, B. E., Houston, T. G., Vose, R. S. (2010). Comprehensive automated quality assurance of daily surface observations. Journal of Applied Meteorology and Climatology 49, 1615-1633.
- Schelter, S., Lange, D., Schmidt, P., Celikel, M., Biessmann, F., Grafberger, A. (2018). Automating large-scale data quality verification. Proceedings of the VLDB Endowment 11(12), 1781-1794.
- Moreau, L., Missier, P. (eds.) (2013). PROV-DM: The PROV Data Model. W3C Recommendation.
- CF Conventions and Metadata, version 1.8 (cfconventions.org); DataCite Metadata Schema 4.x (datacite.org); Zarr specification v3 (zarr-specs.readthedocs.io); Apache Parquet format documentation.

> Items not marked "checked" were cited from memory and still need verification against the original source before publishing.

## 12. Change log

| Date | Version | Change |
|---|---|---|
| 2026-10-23 | v0.1 | Protocol drafted (blueprint); no results yet. |
| 2026-10-23 | v0.2 | System built; decision rules for H1 to H4 written before the final run; baselines, metrics and fault definitions made concrete. Results recorded from one uninterrupted invocation of `uv run python -m experiments.all --fresh` (97 units computed, 0 resumed, 3,062 seconds, `single_invocation: true` in `results/run_meta.json`). |
| 2026-10-23 | v0.2 | Disclosure of earlier partial or discarded runs: (a) two-seed pilots of E1 to E5 were run during development in a scratch directory (not in `results/`) to test the harness; their numbers were seen and discarded, and no parameter was changed afterwards. (b) A pilot E4 run crashed on a Windows "Access is denied" error while renaming a freshly written release directory; a retrying rename helper was added to the library before the final run. (c) A pilot E5 run disagreed between its two runs because the library source was edited while it ran, which the lineage code hash recorded; the pilot was discarded and the code was frozen before the final run. (d) No experiment was rerun after the final run and the code and experiment scripts were not changed after it. |

---

[README](../README.md) · [Client case study](case-study.md) · [Research report](research-report.md)
