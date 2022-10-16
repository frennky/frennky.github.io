# dig

DNS lookup utility

```bash
# dig @ns-server host record-type +short
dig @8.8.8.8 google.com MX +short
```

# ping

Send ICMP ECHO_REQUEST to network hosts

```bash
ping -c 6 -i 10 8.8.8.8
```

|Option|Description|
|:-----|:----------|
|-c <count>|stop after <count> replies|
|-i <interval> |seconds between sending each packet|

# ip

Show / manipulate routing, network devices, interfaces and tunnels

```bash
# ip [ OPTIONS ] OBJECT COMMAND
ip a
# show address of specific interface (with color)
ip -c address show dev eth0
# add ip4 address
ip addr add 192.168.1.10/24 dev eth0
# delete ip4 address
ip add del 192.168.1.10/24 dev eth0
# show statistics of an interface
ip -s link show dev eth0
# change link state to down
ip link set eth0 down
# change link state to up
ip link set eth0 up
# list routes
ip route list
# add default route
ip route add default via 192.168.0.1 dev eth0
# display the route an address will take
ip route get 8.8.8.8
```

|Option|Description|
|:-----|-----------|
|-c|color output|
|-f|protocol family|
|-4|short for -f inet|
|-o|output each record on a single line|
|-s|output statistics|
|-j|output results in json format|
|-p|pretty print json output|
