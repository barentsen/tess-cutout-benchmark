# tess-cutout-benchmark

**How fast can we cut out data from TESS Full Frame Images?**

This repository contains a
[Python notebook](https://github.com/barentsen/tess-cutout-benchmark/blob/master/tess-cutout-benchmark.ipynb)
which measures the time it takes to extract cutouts from TESS Full Frame Images in two different ways:
1. from MAST using the [TESSCut API](https://mast.stsci.edu/tesscut/) accessed via the `astroquery` package;
2. from AWS S3 using the experimental [s3-support branch](https://github.com/spacetelescope/astrocut/pull/44) branch of the `astrocut` package.

The benchmark was run in the following way:
* All benchmarks ran on AWS EC2 using the TIKE platform, which provides 4 virtual cores.
* All cutouts are made for random positions in sectors 30 through 39.
* All cutouts are 10 by 10 pixels in size.


## Results

The table below summarizes the snapshot performance observed in the evening of August 24, 2021 (Pacific Time).
The table shows the wall times that were required to obtain a specific number of random cutouts from MAST using the TESSCut API (`TESSCut`) and from AWS S3 using Astrocut (`TIKE-Astrocut`). The benchmarks were run in parallel using increasing number of processes (`nProc`) on a 4-core EC2 instance provided by TIKE.

| Benchmark                  | nProc | TESSCut         | TIKE-Astrocut
| :------------------------- | ----: | --------------: | ------------:
| 1 cutout                   |     1 |              2s |           2s
| 10 cutouts                 |     1 |             30s |          16s
| 100 cutouts                |     1 |           4m21s |        2m23s
| 1000 cutouts               |     1 |          45m19s |       23m21s
| 10 cutouts (4x parallel)   |     4 |             10s |           5s
| 100 cutouts (4x parallel)  |     4 |           1m39s |          45s
| 1000 cutouts (4x parallel) |     4 |          13m55s |        6m30s
| 10 cutouts (16x parallel)  |   16† |  not supported‡ |           4s
| 100 cutouts (16x parallel) |   16† |  not supported‡ |          37s
| 1000 cutouts (16x parallel)|   16† |  not supported‡ |        5m40s

† TIKE offers 4 virtual cores. As a result, the benchmarks for `nProc=16` saturated the instance and could likely be sped up if a larger instance were used. Monitoring `top` revealed that the TIKE-Astrocut jobs were consistently compute-limited.

‡ The TESSCut API does not currently allow more than 10 simultaneous requests, which is why the entries for `nProc=16` show `not supported`.


## Results ranked by cutouts per second

The table below shows the same results as provided by the table above, but this time ranked by the number of random 10-by-10px cutouts that could be obtained per second.


| Method         | nCutouts | nProc | Cutouts/second
| :------------  | -------: | ----: | -------------:
| TIKE-Astrocut  | 1000     |    16 |   2.94
| TIKE-Astrocut  | 100      |    16 |   2.70
| TIKE-Astrocut  | 1000     |     4 |   2.56
| TIKE-Astrocut  | 10       |    16 |   2.50
| TIKE-Astrocut  | 100      |     4 |   2.22
| TIKE-Astrocut  | 10       |     4 |   2.00
| TESSCut        | 1000     |     4 |   1.20
| TESSCut        | 100      |     4 |   1.01
| TESSCut        | 10       |     4 |   1.00
| TIKE-Astrocut  | 1000     |     1 |   0.71
| TIKE-Astrocut  | 100      |     1 |   0.70
| TIKE-Astrocut  | 10       |     1 |   0.63
| TIKE-Astrocut  | 1        |     1 |   0.50
| TESSCut        | 1        |     1 |   0.50
| TESSCut        | 100      |     1 |   0.38
| TESSCut        | 1000     |     1 |   0.37
| TESSCut        | 10       |     1 |   0.33


## Possible further work

Questions not yet investigated include:
* How does the performance compare to a typical home network?
* What if a more powerful EC2 instance was used? What would the speed-up be when using e.g. 32 cores instead of the 4 provided by TIKE?
* What if the astrocut function were implemented as an AWS Lambda service?
* What if the astrocut cube files were available locally to the instance?
* What if the astrocut cube files were converted to the `zarr` file format first, which provides native support for cloud-based cutouts.
