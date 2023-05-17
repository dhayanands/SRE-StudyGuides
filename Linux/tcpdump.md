## Basics

```bash
## using tcpdump
tcpdump -i eth0 host 192.168.1.3 and port 5060 -n -s 0 -vvv -w /usr/src/dump

tcpdump -i eth0 host  brave.com -n -s 0 -vvv
```

To monitor certain ports:

```bash
# -n is to stop IP name resolution -i is for the interface
sudo tcpdump -n -i eth0 '(port 21 or 22 or 25)'
```

Record traffic looking for a specific host:

```bash
sudo tcpdump -i eth0 -n 'host 192.168.1.2 and (port 21 or 22 or 25)'
```

To write to a file:

```bash
sudo tcpdump -n -i eth0 ip4 -w ipv4.pcap
```

To select hosts from a Pcap file:

```bash
# This assumes the ip and port are together such as: 192.168.2.1.80
sudo tcpdump -n -r server.pcap | cut -d " " -f3 | cut -d "." -f 1-4  | sort -u
```