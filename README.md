# Tctr

Testing distributed systems under hard failures like network partitions and instance termination is critical, but it's also important we test them under [less catastrophic conditions](http://www.bravenewgeek.com/sometimes-kill-9-isnt-enough/) because this is what they most often experience. Tctr is a tool designed to simulate common network problems like latency, bandwidth restrictions, and dropped/reordered/corrupted packets.

It works by wrapping up some system tools in a portable(ish) way. On BSD-derived systems such as OSX, we use tools like `ipfw` and `pfctl` to inject failure. On Linux, we use `iptables` and `tc`. Tctr is merely a thin wrapper around these controls. Windows support may be possible with `wipfw` or even the native network stack, but this has not yet been implemented in Comcast and may be at a later date.

## Installation

```
$ go get github.com/yrong/tctr
```

## Usage

On Linux, Tctr supports several options: device, latency, target/default bandwidth, packet loss, protocol, and port number.

```
$ tctr --device=eth0 --latency=250 --target-bw=1000 --default-bw=1000000 --packet-loss=10% --target-addr=8.8.8.8,10.0.0.0/24 --target-proto=tcp,udp,icmp --target-port=80,22,1000:2000
```

On OSX, Tctr will check for `pfctl` support (as of Yosemite), which supports the same options as above. If `pfctl` is not available, it will use `ipfw` instead, which supports device, latency, target bandwidth, and packet-loss options.

On BSD (with `ipfw`), Tctr currently supports only: device, latency, target bandwidth, and packet loss. 

```
$ tctr --device=eth0 --latency=250 --target-bw=1000 --packet-loss=10%
```

This will add 250ms of latency, limit bandwidth to 1Mbps, and drop 10% of packets to the targetted (on Linux) destination addresses using the specified protocols on the specified port numbers (slow lane). The default bandwidth specified will apply to all egress traffic (fast lane). To turn this off, run the following:

```
$ tctr --stop
```

By default, Tctr will determine the system commands to execute, log them to stdout, and execute them. The `--dry-run` flag will skip execution.


## Network Condition Profiles

Here's a list of network conditions with values that you can plug into Comcast. Please add any more that you may come across.

Name | Latency | Bandwidth | Packet-loss
:-- | --: | --: | --:
GPRS (good) | 500 | 50 | 2
EDGE (good) | 300 | 250 | 1.5
3G/HSDPA (good) | 250 | 750 | 1.5
DIAL-UP (good) | 185 | 40 | 2
DSL (poor) | 70 | 2000 | 2
DSL (good) | 40 | 8000 | 0.5
WIFI (good) | 40 | 30000 | 0.2
Satellite | 1500 | - | 0.2
