# Troubleshooting Port Connectivity Between Linux Network Namespaces

Linux network namespaces provide isolated networking stacks, useful for containers, testing, and custom networking topologies. However, when experimenting with namespaces, you may often need to verify whether a port is reachable between them. 

This guide walks through practical steps to test and troubleshoot port connectivity between two namespaces.

### 1. Verify Basic Connectivity with Ping
Before checking ports, ensure the namespaces can communicate at the IP layer.
```bash
# From ns1 to ns2
ip netns exec ns1 ping -c 3 10.0.0.2
```
If ping fails, the issue is at the link, IP, or firewall level.

### 2. Run a Server in One Namespace

Using Python (HTTP server):
```bash
# Starts a simple built-in HTTP server
ip netns exec ns2 python3 -m http.server 8080
```
You need a service listening on a port inside one namespace.

Using Netcat (TCP):
```bash
# Inside ns2, listen on TCP port 8080
ip netns exec ns2 nc -l -p 8080
```

### 3. Test Connectivity from the Other Namespace
From ns1, try connecting to the service running in ns2.
```bash
ip netns exec ns1 nc -vz 10.0.0.2 8080
```

Alternative using Telnet:
```bash
ip netns exec ns1 telnet 10.0.0.2 8080
```

If ns2 has an IP 10.0.0.2 assigned to its interface, then from ns1 you can test:
```bash
ip netns exec ns1 curl http://10.0.0.2:8080
```

### 4. Confirm Listening Ports with ss or netstat

Inside ns2, confirm that a service is actually listening:`
```bash
ip netns exec ns2 ss -lntp
```

### 5. Testing UDP Connectivity

UDP doesn’t perform handshakes, so testing requires sending actual data.
```bash
# Listener in ns2
ip netns exec ns2 nc -u -l -p 5000


# Client in ns1
ip netns exec ns1 nc -u 10.0.0.2 5000
```

## Additional Notes
- ``ping`` → test network reachability.
- ``nc -vz`` / ``telnet`` → test TCP port connectivity.
- ``ss`` → confirm listening ports.
- ``nc -u ``→ test UDP.