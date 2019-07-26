# nginx-echo-headers

A simple tool for debugging HTTP requests. nginx will return all request headers and body to the client.

Forked this from brndmtthws/nginx-echo-headers and added a customer header variable that can be used to simulate web server latency or processing time for load tests. See examples below.

For the Docker image, see https://hub.docker.com/r/brndnmtthws/nginx-echo-headers/

Try running it like so:

```ShellSession
$ docker run -p 8080:8080 brndnmtthws/nginx-echo-headers
Unable to find image 'brndnmtthws/nginx-echo-headers:latest' locally
latest: Pulling from brndnmtthws/nginx-echo-headers
88286f41530e: Pull complete
7c2e0e2a8099: Pull complete
fe86df227e07: Pull complete
Digest: sha256:177eccf79ee22074a9285341ea61e0a7864023a0a40b115a693256984821328f
Status: Downloaded newer image for brndnmtthws/nginx-echo-headers:latest
2019/01/24 13:55:52 [notice] 1#1: using the "epoll" event method
2019/01/24 13:55:52 [notice] 1#1: openresty/1.11.2.5
2019/01/24 13:55:52 [notice] 1#1: built by gcc 6.3.0 (Alpine 6.3.0)
2019/01/24 13:55:52 [notice] 1#1: OS: Linux 4.9.125-linuxkit
2019/01/24 13:55:52 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2019/01/24 13:55:52 [notice] 1#1: start worker processes
2019/01/24 13:55:52 [notice] 1#1: start worker process 7
2019/01/24 13:56:02 [info] 7#7: *1 client 172.17.0.1 closed keepalive connection
```

Then test it (this example uses [httpie](https://httpie.org/)):

```ShellSession
$ http localhost:8080
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/plain
Date: Thu, 24 Jan 2019 13:56:02 GMT
Server: openresty/1.11.2.5
Transfer-Encoding: chunked

GET / HTTP/1.1
Host: localhost:8080
User-Agent: HTTPie/1.0.2
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
```

Example POST request with delay variable (in seconds). This will cause nginx to sleep for 3 seconds and then respond.

```ShellSession
$ http POST localhost:8080 delay-seconds:3

HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/plain
Date: Fri, 26 Jul 2019 16:39:30 GMT
Server: openresty/1.11.2.5
Transfer-Encoding: chunked
```

An example load test using [rakyll/hey](https://github.com/rakyll/hey) in a docker image ([moosebeandev/hey](https://hub.docker.com/r/moosebeandev/hey))

`-c Number of concurrent connections. Simulating c number of users connecting every 3 seconds in below example.`

`-z Number of seconds to run test.`

```ShellSession
# Running the load test locally
$ docker run --network=host moosebeandev/hey -z 12s -c 100 -m POST -H "delay-seconds: 3" http://localhost:8080/

# Running against a remote host that has echo headers running
$ docker run moosebeandev/hey -z 12s -c 100 -m POST -H "delay-seconds: 3" http://nginx-echo-headers.example.com:8080/

Summary:
  Total:        12.0474 secs
  Slowest:      3.0140 secs
  Fastest:      2.9999 secs
  Average:      3.0040 secs
  Requests/sec: 33.2022


Response time histogram:
  3.000 [1]     |
  3.001 [22]    |■■■■■■■
  3.003 [108]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  3.004 [126]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  3.005 [70]    |■■■■■■■■■■■■■■■■■■■■■■
  3.007 [21]    |■■■■■■■
  3.008 [21]    |■■■■■■■
  3.010 [25]    |■■■■■■■■
  3.011 [0]     |
  3.013 [1]     |
  3.014 [5]     |■■


Latency distribution:
  10% in 3.0015 secs
  25% in 3.0024 secs
  50% in 3.0036 secs
  75% in 3.0047 secs
  90% in 3.0079 secs
  95% in 3.0090 secs
  99% in 3.0137 secs

Details (average, fastest, slowest):
  DNS+dialup:   0.0004 secs, 2.9999 secs, 3.0140 secs
  DNS-lookup:   0.0001 secs, 0.0000 secs, 0.0037 secs
  req write:    0.0000 secs, 0.0000 secs, 0.0019 secs
  resp wait:    3.0033 secs, 2.9998 secs, 3.0072 secs
  resp read:    0.0000 secs, 0.0000 secs, 0.0013 secs

Status code distribution:
  [200] 400 responses
```

