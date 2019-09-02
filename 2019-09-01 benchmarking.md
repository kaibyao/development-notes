Ran into an issue while creating benchmarks to test `rust-postgres-rest-actix`: I was getting non-200/300 responses.

```plaintext
~/D/g/rust-postgres-rest-benchmarks ❯❯❯ wrk -t8 -c125 -d15s "http://127.0.0.1:3000/sibling?columns=name,parent_id.parent_id.name,sibling_id.name"
Running 15s test @ http://127.0.0.1:3000/sibling?columns=name,parent_id.parent_id.name,sibling_id.name
  8 threads and 125 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   237.13ms  356.48ms   1.98s    80.67%
    Req/Sec     2.55k     1.45k   13.44k    73.29%
  297788 requests in 15.02s, 83.21MB read
  Non-2xx or 3xx responses: 297776
Requests/sec:  19829.23
Transfer/sec:      5.54MB
```

It seems that in order to troubleshoot the issue, I need to run benchmarks using k6 so that
I can get the actual responses and troubleshoot from there.

I suspect the issue might have to do with my postgres configuration.
Since I'm using the default postgres settings from the docker image,
I'm assuming that it allows at most 100 simultaneous connections.
But I need to know for sure.

I also wonder if there's a read/write lock error on my stats cache.

Hopefully I will uncover the issue.
