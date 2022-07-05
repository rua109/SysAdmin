# SysAdmin

## NFTables

To list all tables do 
```
nft list tables
```

To show all rulesets do
```
nft list ruleset
```

To import a nft file do
```
nft -f <filename>
```

Use the following structure for creating a firewall
```
table ip firewall
flush table ip firewall
delete table ip firewall

table ip firewall {
  chain prert.nat {
    type nat hook prerouting priority dstnat;
  }
  chain pstrt.nat {
    type nat hook postrouting priority srcnat;
  }
  chain out.nat {
    type nat hook output priority 0;
  }
  chain in.filter {
    type filter hook input priority filter;
  }
  chain out.filter {
    type filter hook output priority filter;
  }
  chain fwd.filter {
    type filter hook forward priority filter;
  }
}
```
### NAT
The following example shows a host `forwarding` rule typically used
```
chain prert.nat {
  type nat hook prerouting priority dstnat;
  ip daddr $wan_ip tcp dport $port counter dnat $machine_ip 
}
chain pstrt.nat {
  type nat hook postrouting priority srcnat;
  ip saddr $internal_network counter snat $wan_ip
}
```

### Tracing
The following example shows how to trace whether packets are coming to preroute
```
table ip firewall_trace
flush table ip firewall_trace
delete table ip firewall_trace

table ip firewall_trace {
  chain prert.nat {
    type nat hook prerouting priority dstnat - 1;
    meta nftrace set 1
  }
}
```

To actually do the tracing do -
```
nft monitor trace
```
```
