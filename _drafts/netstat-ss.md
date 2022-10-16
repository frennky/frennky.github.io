# netstat and ss

Print  network connections, routing tables, interface statistics etc.

```bash
netstat -tulpn
```

| Flag | Description |
|:-----|:------------|
|-t|tcp|
|-u|udp|
|-l|display listening server sockets|
|-p|display PID/Program name for sockets|
|-n|don't resolve names|

Similar output can be printed with `ss` command.

```bash
ss -tulp
```