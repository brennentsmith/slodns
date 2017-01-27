# slodns
Slodns is a forwarding DNS server to simulate slow, jittery, or problematic DNS implementations. It is able to add a uniform or dynamic delay upon DNS datagrams, or drop them at a pre-defined rate. 

```
$ dig speedtest.net @8.8.8.8 | grep "Query time"
;; Query time: 12 msec

$ slodns 
Request: speedtestnet
Response: speedtestnet delay 1000 ms

$ dig speedtest.net @127.0.0.1 -p 5053 | grep "Query time"
;; Query time: 1028 msec
```

## Installation

### Requrirements

Python 2.7 is required to run slodns. No external python resources are required. 

Root is *not* required if you run on the default port. If you want to run on port 53, then you will need to run as root or use iptables. 

### Supported Platforms

slodns has been tested on the following platforms:

1. MacOS 10.12 (Sierra)
2. Debian 7/8
3. Ubuntu 14.04/16.04
4. Freebsd 9/10
5. PfSense 2.3

It will probably work on other platforms as well, feel free to let me know!

### Installation via Git

```sh
wget https://raw.githubusercontent.com/brennentsmith/slodns/master/slodns
chmod +x slodns
```

## Usage

```
usage: A forwarding DNS server to simulate slow, jittery, or problematic DNS implementations.
       [-h] [-i INTERFACE] [-p PORT] [-u UPSTREAM_SERVER]
       [-q UPSTREAM_SERVER_PORT] [-d DELAY] [-l LOSS] [-j JITTER]

optional arguments:
  -h, --help            show this help message and exit
  -i INTERFACE, --interface INTERFACE
                        interface for server to bind to. Defaults to
                        localhost.
  -p PORT, --port PORT  port for server to listen on. Defaults to 5053.
  -u UPSTREAM_SERVER, --upstream-server UPSTREAM_SERVER
                        upstream nameserver. Defaults to 8.8.8.8.
  -q UPSTREAM_SERVER_PORT, --upstream-server-port UPSTREAM_SERVER_PORT
                        upstream nameserver port. Defaults to 53.
  -d DELAY, --delay DELAY
                        delay of requests in milliseconds. Defaults to 1000ms.
  -l LOSS, --loss LOSS  percent of requests which will be 'lost'. Defaults to
                        0
  -j JITTER, --jitter JITTER
                        quantity of random variation +/- base in ms. Defaults
                        to 0 (no jitter). Should not be larger than delay.
```
### Examples

Share slodns to machines on the LAN on port 53.
```
$ sudo slodns -i 0.0.0.0 -p 53
```
Use a local DNS server with a 2 second simulated delay and 0.5s jitter.
```
$ slodns -u 192.168.2.1 -d 2000 -j 500
```
Simulate a lossy network with 500ms delay, 100ms jitter (400 - 600ms), and 25% request loss. 
```
$ slodns -u 192.168.2.1 -d 500 -j 100 -l 25
```
## Running on Privileged Ports
As binding to ports <= 1024 require root execution, you have two choices for slodns. The first is to use `sudo` and run  slodns as root. The other option is to enable port forwarding within the kernel and use iptables - an example rule is below:
```
iptables -t nat -A PREROUTING -s 127.0.0.1 -p tcp --dport 53 -j REDIRECT --to 5053`
iptables -t nat -A OUTPUT -s 127.0.0.1 -p tcp --dport 53 -j REDIRECT --to 5053`
```