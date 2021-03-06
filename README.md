# tess-cutout-benchmark

**How fast can we cut out data from TESS Full Frame Images?**

This repository contains a
[Python notebook](https://github.com/barentsen/tess-cutout-benchmark/blob/master/tess-cutout-benchmark.ipynb)
which measures the time it takes to extract cutouts from TESS Full Frame Images in two different ways:
1. from MAST using the [TESSCut API](https://mast.stsci.edu/tesscut/) accessed via the `astroquery` package running on TIKE;
2. from AWS S3 using the experimental [s3-support branch](https://github.com/spacetelescope/astrocut/pull/44) branch of the `astrocut` package running on TIKE;
3. from AWS S3 using `astrocut` running on Geert's home WiFi.

The benchmark was run in the following way:
* All benchmarks ran on AWS EC2 instances accessed via the TIKE platform, which provides 4 virtual cores.
* All cutouts were made for random positions in sectors 30 through 39.
* All cutouts were 10 by 10 pixels in size.


## Results

The table below shows the snapshot performance observed in the evening of August 24, 2021 (Pacific Time).
The table shows the wall times that were required to obtain a given number of random cutouts from MAST using the TESSCut API running on TIKE (`TESSCut`), from AWS S3 using Astrocut running on TIKE (`TIKE-Astrocut`), and from AWS S3 using Astrocut running on Geert's home WiFi (`Local-Astrocut`). The benchmarks were run in parallel using an increasing number of simultaneous processes (`nProc`).

| Benchmark                  | nProc | TESSCut         | TIKE-Astrocut | Local-Astrocut
| :------------------------- | ----: | --------------: | ------------: | -------------:
| 1 cutout                   |     1 |              2s |           2s  |           15s
| 10 cutouts                 |     1 |             30s |          16s  |         1m06s
| 100 cutouts                |     1 |           4m21s |        2m23s  |         9m54s
| 1000 cutouts               |     1 |          45m19s |       23m21s  |           TBD
| 10 cutouts (4x parallel)   |     4 |             10s |           5s  |           24s
| 100 cutouts (4x parallel)  |     4 |           1m39s |          45s  |         3m26s
| 1000 cutouts (4x parallel) |     4 |          13m55s |        6m30s  |           TBD
| 10 cutouts (16x parallel)  |   16??? |  not supported??? |           4s  |           19s
| 100 cutouts (16x parallel) |   16??? |  not supported??? |          37s  |         2m31s
| 1000 cutouts (16x parallel)|   16??? |  not supported??? |        5m40s  |           TBD

??? TIKE offers 4 virtual cores. As a result, the benchmarks for `nProc=16` saturated the instance and could likely be sped up if a larger instance were used; monitoring `top` revealed that the TIKE-Astrocut jobs were consistently compute-limited.

??? The TESSCut API does not currently allow more than 10 simultaneous requests, hence its entries for `nProc=16` show `not supported`.


## Results ranked by cutouts per second

The table below shows the same results as provided by the table above, but this time ranked by the number of cutouts that could be obtained per second.


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
| Local-Astrocut | 100      |    16 |   0.66
| TIKE-Astrocut  | 10       |     1 |   0.63
| Local-Astrocut | 10       |    16 |   0.53
| TIKE-Astrocut  | 1        |     1 |   0.50
| TESSCut        | 1        |     1 |   0.50
| Local-Astrocut | 100      |     4 |   0.49
| Local-Astrocut | 10       |     4 |   0.42
| TESSCut        | 100      |     1 |   0.38
| TESSCut        | 1000     |     1 |   0.37
| TESSCut        | 10       |     1 |   0.33
| Local-Astrocut | 100      |     1 |   0.17
| Local-Astrocut | 10       |     1 |   0.15
| Local-Astrocut | 1        |     1 |   0.06


## Possible further work

Questions not yet investigated include:
* What if a more powerful EC2 instance was used? What would the speed-up be when using e.g. 32 cores instead of the 4 provided by TIKE?
* What if the astrocut function were implemented as an AWS Lambda service?
* What if the astrocut cube files were available locally to the instance?
* What if the astrocut cube files were converted to the `zarr` file format first, which provides native support for cloud-based cutouts and may require less compute power on the client side.
