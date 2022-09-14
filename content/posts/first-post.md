---
title: "Troubleshooting a mysterious Python test failure"
date: 2022-09-13T20:31:01+08:00
draft: false
---

**Context**

At my day job my colleagues and I develop a [data gathering and visualisation platform](https://www.ftpsolutions.com.au/products/ims/) that
has a fair bit of [Python](https://www.python.org/) behind the scenes.

We test a lot of this Python using [pytest](https://docs.pytest.org/en/7.1.x/) and all our tests are run by a large locally hosted instance
of [TeamCity](https://www.jetbrains.com/teamcity/).

A test run involves TeamCity executing a [bash](https://www.gnu.org/software/bash/) script responsible for setting up any test
dependencies (usually [Docker](https://www.docker.com/) containers) and then executing the test itself (also usually a Docker container).

**Problem**

My interest was piqued by one of my colleagues trying to understand why a
particular long-running  [end-to-end test](https://www.testim.io/blog/end-to-end-testing-guide/) had suddenly started failing part-way
through with seemingly no relevant changes to any nearby code.

TeamCity gave us some clear information as to when the trouble started:

![Image 1](/posts/first-post/image-1.png)

The TeamCity [build log](https://www.jetbrains.com/help/teamcity/build-log.html#Viewing+Build+Log) doesn't show a lot beyond the test
execution suddenly stopping with an ominous `Killed` message:

![Image 2](/posts/first-post/image-2.png)

Enabling "Verbose" mode in the build log output for TeamCity gives us a little more context and clarifies that the test container died
suddenly:

![Image 3](/posts/first-post/image-3.png)

**First thoughts**

Leaning on some prior knowledge, I'm reasonably confident of the following:

- The [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout)) of a process will read `Killed` when some other
  process kills a process (such as the [kill](https://man7.org/linux/man-pages/man2/kill.2.html) command)
    - This was observed in the build log
- TeamCity will show related `stop` and possible `kill` Docker events preceding a `die` Docker event if the container was request to stop (
  such as using the [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) command)
    - This _was not_ observed in the build log

When things get killed at random, my mind immediately goes to
the [Linux OOM Killer](https://en.wikipedia.org/wiki/Out_of_memory#Out_of_memory_management) (a function of the Linux kernel dedicated to
identifying and killing memory hogging processes).

Processes running inside Docker containers on Linux are [namespaced](https://en.wikipedia.org/wiki/Linux_namespaces) to achieve what feels
like isolation to the process in question; to the OOM Killer though, a process is just a process and if it's using too much memory it's
going to get killed to save the system.

**Digging deeper**

Unfortunately the TeamCity runner VMs had all been rebooted (redeployed in fact) since the last failing test run as part of some ongoing
improvements to our [CI](https://en.wikipedia.org/wiki/Continuous_integration) system and so any
incriminating [dmesg](https://man7.org/linux/man-pages/man1/dmesg.1.html) logs were long gone.

Fortunately we have some long-term metrics as part of our [Grafana](https://grafana.com/) / [Prometheus](https://prometheus.io/)
/ [VictoriaMetrics](https://victoriametrics.com/) deployment that shows some OOM Killer activity for that particular TeamCity runner VM:

![Image 4](/posts/first-post/image-4.png)

While not exactly a smoking gun, it definitely suggests that we run out of memory on that (and probably all) TeamCity runner VMs from
time-to-time.

Here are the specs for that VM according to our [Proxmox](https://www.proxmox.com/en/) cluster:

![Image 5](/posts/first-post/image-5.png)

You'll have to trust me on this next statement as I didn't grab a screenshot: there was no swap configured- this is relevant, as
depending on the amount of memory needed and the amount our test was likely to be affected by bad performance, having swap configured could
potentially absorb memory leaks and permit our test to pass.

**Recreating the problem**

With a strong hypothesis formed, it's time to set about proving it- not wanting to spend much effort on potentially throwaway scaffolding, I
decided just to run the test locally (in Docker, but for macOS) and just watch its memory usage in the meantime.

That looked a bit like this in [htop](https://htop.dev/) (note the memory percentage increasing, excuse the GIF encoding):

![Video 1](/posts/first-post/video-1.gif)

After some quick Googling on how best to do memory profiling on a Python process without having to edit the actual code I came
across [memray](https://github.com/bloomberg/memray) which appealed to me for two reasons:

- I could [use it as a wrapper](https://github.com/bloomberg/memray#usage) to execute my tests
- It has had [recent commits](https://github.com/bloomberg/memray/commits/main)

Installing and running it was a trifle:

```shell
pip install memray

python -m memray run -o output.bin \
  -m pytest -vv e2e/tests/socket_api_vs_db_stat_import_server_test.py
```

After the test completed, `memray` tells us what to run to generate a [flamegraph](https://github.com/brendangregg/FlameGraph) that we can
then open:

```shell
python -m memray flamegraph output.bin

open memray-flamegraph-output.html
```

The result is quite interesting:

![Image 6](/posts/first-post/image-6.png)

We can see that the largest 3 items are all related to logging; there's nothing particularly interesting about the way we do logging (but we
do a lot of it)- the items in question are some of the larger log entries (e.g. dictionaries
being [repr'd](https://docs.python.org/3/library/functions.html#repr)).

If we check the summary to see the overall damage:

![Image 7](/posts/first-post/image-7.png)

Oh it's bad.

It's at this point I recall
that `pytest` [keeps tracks of all Python loggers](https://docs.pytest.org/en/7.1.x/how-to/logging.html#how-to-manage-logging) and includes
their messages in the test output (something I've intentionally disabled before in favour of live-logging to get context on long-running
tests)- could it be that the sheer volume of log messages and the extended duration of the tests is using up all the RAM?

**Testing the hypothesis**

This is simple enough- we just have to re-run the test with the log capturing disabled (you probably don't need all of these flags, but my
initial attempts with just `--capture=no` didn't work so I went scorched earth with the flags):

```shell
python -m memray run -o output.bin \
  -m pytest -vv \
  --capture=no --show-capture=no --log-level=CRITICAL \
  --log-cli-level=CRITICAL -o junit_logging=no \
  e2e/tests/socket_api_vs_db_stat_import_server_test.py

python -m memray flamegraph output.bin

open memray-flamegraph-output.html
```

The flamegraph shape more closely represents the work being done (lots
of [serialisation of data between Go and Python](https://github.com/ftpsolutions/gomssql-python/)):

![Image 8](/posts/first-post/image-8.png)

And the summary reflects far less memory usage:

![Image 9](/posts/first-post/image-9.png)

**Resolution**

With a pretty clear cut smoking gun, we can set about fixing the problem; here are a few options available:

- Inside our platform
    - Remove or edit those large log entries (if they're not useful)
    - Stub out the logger with a dummy logger (not actually a logger for `pytest` purposes but feels like one to our code)
- Outside our platform
    - Change to another Python test runner
    - Just disable log recording (as we did to test the hypothesis)

For this particular situation, the logs caught by `pytest` are not that useful- they're usually just logs showing repeated RPC failures or
empty query / request results while waiting for them to be non-empty, so it's acceptable to simply check-in the extra `pytest` flags for
disabling log recording and call it a day!
