# SysAdmin

## Burning a iso disk

Plug your USB drive into the computer. Type the following command
```
lsblk
```

Unmount the USB drive
```
sudo umount /dev/sdbx
```

Check if the USB is unmounted by executing `lsblk` again

Burn the ISO to the USB drive
```
sudo dd bs=4M if=img.iso of=/dev/sdbx status=progress oflag=sync
```
Here is what the flags mean -
- `dd` is the command that copies and converts data
- `bs=4M` sets the block size to `4MB` for efficient data transfer
- `if=img.iso` specifies your ISO file as the input file
- `of=/dev/sdbx` specifies USB drive as output file
- `status=progress` shows progress bar during the process
- `oflag=sync` ensures that the command doesn't finish until the USB drive has received all the data successfully.

## NFTables

### Routing

Following figure shows how routing is done by NFTables (src: https://wiki.nftables.org/wiki-nftables/index.php/Netfilter_hooks)

![Routing](/assets/images/nf-hooks.png)


### Basic commands

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

The **priority** parameter accepts a signed integer value or a standard priority name which specifies the order in which chains with same **hook** value are traversed. The ordering is ascending, i.e. lower priority values have precedence over higher ones.

| Name     | Value | Families                   | Hooks       |
|----------|-------|----------------------------|-------------|
|  raw     | -300  | ip, ip6, inet              | all         |
| mangle   | -150  | ip, ip6, inet              | all         |     
| dstnat   | -100  | ip, ip6, inet              | prerouting  |
| filter   | 0     | ip, ip6, inet, arp, netdev | all         |
| security | 50    | ip, ip6, inet              | all         |
| srcnat   | 100   | ip, ip6, inet              | postrouting |



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
