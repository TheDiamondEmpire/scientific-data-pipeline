## E1

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

## E2

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

## E3

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

Read latency (seconds; median and bootstrap 95% CI):

| Dataset | Format | Query | Kind | n | Median s | 95% CI | Scan MB/s |
|---|---|---|---|---|---|---|---|
| grid | csv | scan | cold | 12 | 1.1377 | [0.9269, 1.6903] | 8.7 |
| grid | csv | scan | warm | 24 | 1.0774 | [0.9650, 1.2565] | 9.2 |
| grid | csv | series | cold | 12 | 1.3643 | [1.2249, 1.7769] |  |
| grid | csv | series | warm | 24 | 1.1370 | [1.1094, 1.3717] |  |
| grid | csv | slice | cold | 12 | 1.6497 | [1.1785, 2.1776] |  |
| grid | csv | slice | warm | 24 | 1.3411 | [1.1504, 1.5508] |  |
| grid | csv_gz | scan | cold | 12 | 1.6331 | [1.4728, 1.8131] | 6.1 |
| grid | csv_gz | scan | warm | 24 | 1.5595 | [1.3102, 1.6530] | 6.4 |
| grid | csv_gz | series | cold | 12 | 2.0684 | [1.9468, 2.6279] |  |
| grid | csv_gz | series | warm | 24 | 1.9742 | [1.6525, 2.2060] |  |
| grid | csv_gz | slice | cold | 12 | 1.7881 | [1.5567, 2.4834] |  |
| grid | csv_gz | slice | warm | 24 | 1.5885 | [1.4835, 1.9101] |  |
| grid | parquet | scan | cold | 12 | 0.1596 | [0.1319, 0.2057] | 62.4 |
| grid | parquet | scan | warm | 24 | 0.0264 | [0.0242, 0.0308] | 377.6 |
| grid | parquet | series | cold | 12 | 0.1627 | [0.1563, 0.2633] |  |
| grid | parquet | series | warm | 24 | 0.0409 | [0.0316, 0.0480] |  |
| grid | parquet | slice | cold | 12 | 0.0856 | [0.0771, 0.1503] |  |
| grid | parquet | slice | warm | 24 | 0.0116 | [0.0097, 0.0140] |  |
| grid | zarr | scan | cold | 12 | 0.2232 | [0.1967, 0.2736] | 44.6 |
| grid | zarr | scan | warm | 24 | 0.0158 | [0.0144, 0.0175] | 628.6 |
| grid | zarr | series | cold | 12 | 0.0776 | [0.0658, 0.1152] |  |
| grid | zarr | series | warm | 24 | 0.0266 | [0.0240, 0.0310] |  |
| grid | zarr | slice | cold | 12 | 0.0694 | [0.0668, 0.0801] |  |
| grid | zarr | slice | warm | 24 | 0.0184 | [0.0168, 0.0241] |  |
| real | csv | scan | cold | 12 | 0.2886 | [0.2326, 0.3919] | 0.4 |
| real | csv | scan | warm | 24 | 0.1681 | [0.1269, 0.2179] | 0.7 |
| real | csv | series | cold | 12 | 0.3144 | [0.1986, 0.4271] |  |
| real | csv | series | warm | 24 | 0.1741 | [0.1442, 0.2502] |  |
| real | csv | slice | cold | 12 | 0.2740 | [0.2238, 0.3914] |  |
| real | csv | slice | warm | 24 | 0.1624 | [0.1427, 0.1972] |  |
| real | csv_gz | scan | cold | 12 | 0.3088 | [0.2518, 0.4124] | 0.4 |
| real | csv_gz | scan | warm | 24 | 0.1690 | [0.1502, 0.2171] | 0.7 |
| real | csv_gz | series | cold | 12 | 0.2791 | [0.2365, 0.3763] |  |
| real | csv_gz | series | warm | 24 | 0.1890 | [0.1679, 0.2348] |  |
| real | csv_gz | slice | cold | 12 | 0.4190 | [0.2957, 0.4997] |  |
| real | csv_gz | slice | warm | 24 | 0.1601 | [0.1450, 0.1889] |  |
| real | parquet | scan | cold | 12 | 0.1651 | [0.0964, 0.2199] | 0.7 |
| real | parquet | scan | warm | 24 | 0.0073 | [0.0065, 0.0093] | 15.8 |
| real | parquet | series | cold | 12 | 0.1177 | [0.0908, 0.1589] |  |
| real | parquet | series | warm | 24 | 0.0093 | [0.0075, 0.0114] |  |
| real | parquet | slice | cold | 12 | 0.1160 | [0.0824, 0.1602] |  |
| real | parquet | slice | warm | 24 | 0.0081 | [0.0070, 0.0110] |  |
| real | zarr | scan | cold | 12 | 0.0769 | [0.0679, 0.0843] | 1.5 |
| real | zarr | scan | warm | 24 | 0.0017 | [0.0016, 0.0020] | 67.5 |
| real | zarr | series | cold | 12 | 0.0751 | [0.0708, 0.0821] |  |
| real | zarr | series | warm | 24 | 0.0160 | [0.0147, 0.0184] |  |
| real | zarr | slice | cold | 12 | 0.0620 | [0.0545, 0.0746] |  |
| real | zarr | slice | warm | 24 | 0.0059 | [0.0052, 0.0069] |  |

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

## E4

| Stations | Metric | n pairs | Off median s | On median s | Overhead % (median) | 95% CI | Median diff s | Wilcoxon p | CI upper < 5% |
|---|---|---|---|---|---|---|---|---|---|
| 2 | wall_s | 12 | 5.29 | 7.38 | 30.7 | [19.2, 55.6] | 1.74 | 0.0005 | no |
| 2 | cpu_s | 12 | 1.86 | 2.57 | 42.9 | [36.0, 45.9] | 0.79 | 0.0005 | no |
| 4 | wall_s | 12 | 3.36 | 5.05 | 51.6 | [18.8, 61.6] | 1.59 | 0.0005 | no |
| 4 | cpu_s | 12 | 2.34 | 3.30 | 44.5 | [36.8, 50.1] | 0.97 | 0.0005 | no |
| 8 | wall_s | 12 | 4.80 | 6.30 | 29.7 | [19.6, 45.7] | 1.34 | 0.0015 | no |
| 8 | cpu_s | 12 | 3.35 | 4.62 | 32.6 | [26.6, 41.1] | 1.13 | 0.0005 | no |

## E5

| Artefact | n clean runs | Distinct hashes | Agreement with modal | 95% CI (Wilson) | Expected | Perturbed-input control differs |
|---|---|---|---|---|---|---|
| release_tree | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| curated_tree | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| quarantine_tree | 10 | 1 | 10/10 | [0.72, 1.00] | identical | False |
| lineage_core | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| all_deterministic_files | 10 | 1 | 10/10 | [0.72, 1.00] | identical | True |
| run_log_volatile | 10 | 10 | 1/10 | [0.02, 0.40] | differ (volatile control) | True |

Files compared per run: 180. Varied between runs: PYTHONHASHSEED, thread cap (1 or 2), working directory path, current directory.
