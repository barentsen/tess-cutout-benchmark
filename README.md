# tess-cutout-benchmark

**How fast can we cut out data from TESS Full Frame Images?**

This repository contains a
[Python notebook](https://github.com/barentsen/tess-cutout-benchmark/blob/master/tess-cutout-benchmark.ipynb)
which measures the time it takes to extract cutouts from TESS Full Frame Images in two different ways:
1. from MAST using the [TESSCut API](https://mast.stsci.edu/tesscut/) accessed via the `astroquery` package;
2. from AWS S3 using the experimental [s3-support](https://github.com/spacetelescope/astrocut/pull/44) branch of the `astrocut` package.

The benchmark was run in the following way:
* All benchmarks ran on AWS EC2 using the TIKE platform, which provides 4 virtual cores.
* All cutouts are made for random positions in sectors 30 through 39.
* All cutouts are 10 by 10 pixels in size.


## Results

The table below shows the typical results obtained at a single point in time in the evening of August 24, 2021 (Pacific Time).
The wall times required to obtain the cutouts from MAST via the TESSCut API (`TESSCut`) and from AWS S3 using Astrocut (`TIKE-Astrocut`) are shown. The benchmarks were run in parallel using increasing number of processes (`nProc`) on a single TIKE instance.

| Benchmark               | nProc  | TESSCut | TIKE-Astrocut
| -----------             | ----: | ------: | ---------:
| 1 cutout                |     1 |      2s |         2s
| 10 cutouts              |     1 |     30s |        16s
| 100 cutouts             |     1 |   4m21s |      2m23s
| 1000 cutouts            |     1 |     45m19s |     23m21s
| 10 cutouts (4x parallel)   |     4 |     10s |         5s
| 100 cutouts (4x parallel)  |     4 |   1m39s |        45s
| 1000 cutouts (4x parallel) |     4 |  13m55s |      6m30s
| 10 cutouts (16x parallel)  |    16† |  disallowed‡ |       4s
| 100 cutouts (16x parallel) |    16† |  disallowed‡ |     37s
| 1000 cutouts (16x parallel) |    16† |  disallowed‡ |    5m40s


† TIKE offers 4 virtual cores. As a result, the benchmarks for `nProc=16` saturated the instance and could likely be sped up if a larger instance were used.

‡ The TESSCut API does not currently allow more than 10 simultaneous requests, which is why the entries for `nProc=16` show `disallowed`.


## Further work

Questions not yet investigated include:
* What is the performance if the astrocut function were implemented as an AWS Lambda service?
* What if the astrocut cube files were available locally?
* What if the astrocut cube files were converted to the `zarr` file format first, which provides native support for cloud-based cutouts.
