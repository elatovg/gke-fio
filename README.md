# gke-io

## Deploy it
Deploy the manifests:

```bash
> k apply -f .
pod/pod-sysbench created
persistentvolumeclaim/pvc-io created
```

`exec` into the container:

```bash
> kubectl exec -it pod-sysbench -- /bin/bash
> root@pod-sysbench:/#
```
Then we can run some tests.

## Sysbench test
Simple `sysbench` test:

```bash
root@pod-sysbench:/# cd /data
root@pod-sysbench:/data# sysbench --file-total-size=10G --file-num=10 fileio prepare
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

10 files, 1048576Kb each, 10240Mb total
Creating files for the test...
Extra file open flags: (none)
Creating file test_file.0
Creating file test_file.1
Creating file test_file.2
Creating file test_file.3
Creating file test_file.4
Creating file test_file.5
Creating file test_file.6
Creating file test_file.7
Creating file test_file.8
Creating file test_file.9
10737418240 bytes written in 14.73 seconds (695.28 MiB/sec).
root@pod-sysbench:/data# sysbench --file-total-size=10G --file-test-mode=seqrewr --time=60 --max-requests=0 --threads=10 fileio run
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 10
Initializing random number generator from current time


Extra file open flags: (none)
128 files, 80MiB each
10GiB total file size
Block size 16KiB
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing sequential rewrite test
Initializing worker threads...

Threads started!


File operations:
    reads/s:                      0.00
    writes/s:                     18765.34
    fsyncs/s:                     24039.77

Throughput:
    read, MiB/s:                  0.00
    written, MiB/s:               293.21

General statistics:
    total time:                          60.0163s
    total number of events:              2567792

Latency (ms):
         min:                                    0.01
         avg:                                    0.23
         max:                                  134.99
         95th percentile:                        1.04
         sum:                               596409.85

Threads fairness:
    events (avg/stddev):           256779.2000/1247.15
    execution time (avg/stddev):   59.6410/0.02
root@pod-sysbench:/data# sysbench --file-total-size=10G fileio cleanup
sysbench 1.0.20 (using system LuaJIT 2.1.0-beta3)

Removing test files...
```

## Run a DD test
Or you can run a simple `dd` test:

```bash
root@pod-sysbench:/# dd if=/dev/zero of=/data/file.io bs=1024 count=10M
10485760+0 records in
10485760+0 records out
10737418240 bytes (11 GB, 10 GiB) copied, 39.6902 s, 271 MB/s
```

And then delete the file:

```bash
root@pod-sysbench:/# rm /data/file.io
```

## Clean up
After you done with IO test, just exit out of the container, and delete the deployed resources:

```bash
root@pod-sysbench:/# exit
exit
> k delete -f .
pod "pod-sysbench" deleted
persistentvolumeclaim "pvc-io" deleted
```
