# nftables cheetsheet

`nftables` replaces `iptables` in the newer version of linux dictributions

- [nftables cheetsheet](#nftables-cheetsheet)
  - [Basic Concept](#basic-concept)
  - [Create Tables](#create-tables)
  - [Create Chains](#create-chains)
  - [Add Rules](#add-rules)
  - [Delete Rules](#delete-rules)
  - [List Rules & Chains](#list-rules--chains)
  - [Sets](#sets)
    - [Anonymous Sets](#anonymous-sets)
    - [Named Sets](#named-sets)
  - [Set Intervals](#set-intervals)
  - [Set Concatenations](#set-concatenations)
  - [Verdict Maps](#verdict-maps)
  - [Save and Restore a Ruleset](#save-and-restore-a-ruleset)

## Basic Concept

- nftables manages the rules with tables and chains
- nftables does not pre-create tables and chains
- A `table` is simply a namespace and `chains` are the objects that will contain our firewall rules
- Tables need to be manually created and need to qualify a family: `ip, ip6, inet, arp, bridge, or netdev`
- `inet` means the table will process both ipv4 and ipv6 packets

## Create Tables

`nft list ruleset` list all the rulesets

`nft add table inet my_table` create table

- chains also need to be explicitly created
- When creating the chain we need to specify :
  - what table the chain belongs to
  - the type
  - the hook
  - the priority
- Tables Are Namespaces, so, two tables can create chains, sets, and other objects that have the same name

```bash
nft add table inet table_one
nft add chain inet table_one my_chain
nft add table inet table_two
nft add chain inet table_two my_chain
nft list ruleset
...
table inet table_one {
    chain my_chain {
    }
}
table inet table_two {
    chain my_chain {
    }
}
```

## Create Chains

`nft add chain inet my_table my_filter_chain { type filter hook input priority 0 \; }` simple chain

**Note**: The backslash (\) is necessary so the shell doesn’t interpret the semicolon as the end of the command

- Chains can also be created without specifying a hook
- Rules can use the jump or goto statements to execute rules in the chain
- This is useful to logically separate rules or to share a subset of rules that would otherwise be duplicated

`nft add chain inet my_table my_utility_chain` - chain without hook

## Add Rules

- adding rules

`nft add rule inet my_table my_filter_chain tcp dport ssh accept` - rule to accept ssh
`nft insert rule inet my_table my_filter_chain tcp dport http accept` - rule to accept http

- using `index` to specify where to insert the rule. index of the rule will change everytime when a rule is added or deleted

`nft insert rule inet my_table my_filter_chain index 1 tcp dport nfs accept` - rule to accept nfs
`nft add rule inet my_table my_filter_chain index 0 tcp dport 1234 accept` rule to accept port 1234

Here:

- the rule will process both IPv4 & IPv6 as we have used `inet` family
- `add` verb wil append the rule to end of the chain
- `insert` verb will prepend the rule to head of the chain
- `index` can be used to specify the index no in the list of rules. `add` will insert after the `index` no and `insert` will insert before the `index` no

```bash
## list the ruleset
nft list ruleset

table inet my_table {
    chain my_filter_chain {
    type filter hook input priority 0; policy accept;
    tcp dport http accept
    tcp dport 1234 accept
    tcp dport nfs accept
    tcp dport ssh accept
    }
}
```

- using `handle` to specify where to insert the rule. a rule handle is stable and will not change until the rule is deleted

`nft add rule inet my_table my_filter_chain handle 3 tcp dport 1234 accept`
`nft insert rule inet my_table my_filter_chain handle 2 tcp dport nfs accept`

```bash
## list the ruleset with the handle for rules
nft --handle list ruleset

table inet my_table { # handle 21
    chain my_filter_chain { # handle 1
    type filter hook input priority 0; policy accept;
    tcp dport http accept # handle 3
    tcp dport ssh accept # handle 2
    }
}

## add rule after handle 3
nft add rule inet my_table my_filter_chain handle 3 tcp dport 1234 accept
## add rule before handle 2
nft insert rule inet my_table my_filter_chain handle 2 tcp dport nfs accept

nft --handle list ruleset
table inet my_table { # handle 21
    chain my_filter_chain { # handle 1
    type filter hook input priority 0; policy accept;
    tcp dport http accept # handle 3
    tcp dport 1234 accept # handle 8
    tcp dport nfs accept # handle 7
    tcp dport ssh accept # handle 2
    }
}

## get the handle during time of creation 
nft --echo --handle add rule inet my_table my_filter_chain udp dport 3333 accept

## OUTPUT
add rule inet my_table my_filter_chain udp dport 3333 accept # handle 4
```

## Delete Rules

- Deleting rules is done by using the rule handle similar to the add and insert commands

```bash
## list all rules in ruleset with handle
nft --handle list ruleset

table inet my_table { # handle 21
    chain my_filter_chain { # handle 1
    type filter hook input priority 0; policy accept;
    tcp dport http accept # handle 3
    tcp dport 1234 accept # handle 8
    tcp dport nfs accept # handle 7
    tcp dport ssh accept # handle 2
    }
}

## using hadle to delete rules
nft delete rule inet my_table my_filter_chain handle 8

nft --handle list ruleset

table inet my_table { # handle 21
    chain my_filter_chain { # handle 1
    type filter hook input priority 0; policy accept;
    tcp dport http accept # handle 3
    tcp dport nfs accept # handle 7
    tcp dport ssh accept # handle 2
    }
}
```

## List Rules & Chains

```bash
## List all rules in a given table
nft list table inet my_table

table inet my_table {
    chain my_filter_chain {
        type filter hook input priority 0; policy accept;
        tcp dport http accept
        tcp dport nfs accept
        tcp dport ssh accept
    }
}


## List all rules in a given chain
nft list chain inet my_table my_other_chain

table inet my_table {
    chain my_other_chain {
        udp dport 12345 log prefix "UDP-12345"
    }
}
```

## Sets

`sets` can be useful to match multiple IP addresses, port numbers, interfaces, or any other match criteria

### Anonymous Sets

- Anonymous sets are specified as part of the rule. Used when the sets are not expected to change.
- If the set needs to be changed, the rule need to be replaced, which is a disadvantage and why named set is preferred

```bash
## Allow traffic from 192.168.1.10 and 192.168.1.20
nft add rule inet my_table my_filter_chain ip saddr { 192.168.1.10, 192.168.1.20 } accept

nft list ruleset
table inet my_table {
    chain my_filter_chain {
        type filter hook input priority 0; policy accept;
        tcp dport http accept
        tcp dport nfs accept
        tcp dport ssh accept
        ip saddr { 192.168.1.10, 192.168.1.20 } accept
    }
}

## Allow traffic to http, nfs & ssh
nft add rule inet my_table my_filter_chain tcp dport { http, nfs, ssh } accept
```

### Named Sets

- Named sets are mutable sets
- While creating a named set, we must specify the type of elements they will contain. example types are; ipv4_addr, inet_service, ether_addr
- `@` symbol followed by the set name is how the set in a rule is referenced

```bash
## add a named set to table
nft add set inet my_table my_set { type ipv4_addr \; }

nft list sets

table inet my_table {
    set my_set {
    type ipv4_addr
    }
}

## add element to the set

nft add element inet my_table my_set { 192.168.1.10, 192.168.1.20 }

nft list set inet my_table my_set

table inet my_table {
    set my_set {
    type ipv4_addr
    elements = { 192.168.1.10, 192.168.1.20 }
    }
}

## specifying a set in rule using `@`
nft insert rule inet my_table my_filter_chain ip saddr @my_set drop

nft list chain inet my_table my_filter_chain

table inet my_table {
    chain my_filter_chain {
    type filter hook input priority 0; policy accept;
    ip saddr @my_set drop
    tcp dport http accept
    tcp dport nfs accept
    tcp dport ssh accept
    ip saddr { 192.168.1.10, 192.168.1.20 } accept
    }
}
```

## Set Intervals

- Set intervals are used to specify range of values in a set when using IP addresses
- To use ranges, sets should be created with `interval` flag

```bash
## create set with interval flag
nft add set inet my_table my_range_set { type ipv4_addr \; flags interval \; }

## add ip address range to set
nft add element inet my_table my_range_set  { 10.20.20.0/24 }

nft list set inet my_table my_range_set
table inet my_table {
    set my_range_set {
    type ipv4_addr
    flags interval
    elements = { 10.20.20.0/24 }
    }
}
```

**Note**: The netmask notation was implicitly converted into a range of IP addresses. We could have also used 10.20.20.0-10.20.20.255 to achieve the same effect

## Set Concatenations

- Set Concatenations supports aggregate types and matches
- A set element can contain multiple types and a rule can use the concatenation operator `.` when referencing the set

```bash
## match IPv4 addresses, IP protocols, and port numbers all at once
nft add set inet my_table my_concat_set  { type ipv4_addr . inet_proto . inet_service \; }

nft list set inet my_table my_concat_set

table inet my_table {
    set my_concat_set {
    type ipv4_addr . inet_proto . inet_service
    }
}

## add elements to the list
nft add element inet my_table my_concat_set { 10.30.30.30 . tcp . telnet }

## Using the set in a rule is similar to the name set, but the rule must perform the concatenation
nft add rule inet my_table my_filter_chain ip saddr . meta l4proto . tcp dport @my_concat_set accept

nft list chain inet my_table my_filter_chain

table inet my_table {
    chain my_filter_chain {
    ...
    ip saddr { 10.10.10.123, 10.10.10.231 } accept
    meta nfproto ipv4 ip saddr . meta l4proto . tcp dport @my_concat_set accept
    }
}

## using concatination with inline set
nft add rule inet my_table my_filter_chain ip saddr . meta l4proto . udp dport { 10.30.30.30 . udp . bootps } accept

```

## Verdict Maps

- Verdict maps allow us to perform an action based on packet information, they map match criteria to an action
- Verdict map can be used in order to logically divide the ruleset you with dedicated chains for processing TCP and UDP packets and to steer packets to those chains using a single rule

```bash
## Create tcp & udp chain
nft add chain inet my_table my_tcp_chain
nft add chain inet my_table my_udp_chain

## create verdict map - `vmap`
nft add rule inet my_table my_filter_chain meta l4proto vmap { tcp : jump my_tcp_chain, udp : jump my_udp_chain }

nft list chain inet my_table my_filter_chain
table inet my_table {
    chain my_filter_chain {
    ...
    meta nfproto ipv4 ip saddr . meta l4proto . udp dport { 10.30.30.30 . udp . bootps } accept
    meta l4proto vmap { tcp : jump my_tcp_chain, udp : jump my_udp_chain }
    }
}
```

## Save and Restore a Ruleset

The nftable rules are saved in `/etc/sysconfig/nftables.conf`

```bash
## To save your ruleset
nft list ruleset > /root/nftables.conf

## To restore your ruleset
nft -f /root/nftables.conf

## enable the systemd service and have your rules restored on reboot
systemctl enable nftables
nft list ruleset > /etc/sysconfig/nftables.conf
```
