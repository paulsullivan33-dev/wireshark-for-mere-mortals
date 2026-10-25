# Capture Filters

A capture filter decides which packets
get **recorded at all**. It is applied
by the capture library before a packet
ever reaches disk, and what it rejects
is gone: no filter change tomorrow will
bring it back. That is its power — a
weekend-long capture on a busy network
stays a sane size — and its danger.

The language is not Wireshark's. It is
**BPF syntax**, inherited from libpcap
and tcpdump (chapter 13), and it is much
smaller and dumber than the display
filter language of chapter 6: no field
names like `http.host`, no contains, no
analysis fields. It talks in addresses,
ports, protocols, and byte offsets, and
it must — it runs at wire speed, before
any dissection exists.

In the window, the capture filter goes
in the box on the welcome screen (or in
Capture → Options). In tshark and
dumpcap it follows `-f`:

```bash
tshark -i eth0 -f "tcp port 8080" -w web.pcapng
```

## The shape of the language

| Piece | Example | Means |
|---|---|---|
| Host | `host 192.168.1.20` | Either direction, that address |
| Network | `net 192.168.1.0/24` | A whole subnet |
| Port | `port 53` | Either direction, either transport |
| One protocol | `tcp`, `udp`, `icmp`, `arp` | That protocol only |
| Direction | `src host …`, `dst port …` | One end only |
| Logic | `tcp and not port 22` | `and`, `or`, `not`, parentheses |

Recipes that cover most honest needs:

```text
host 192.168.1.20                          one machine only
port 53                                    all DNS, both transports
tcp port 80 or tcp port 443                the web, encrypted and not
not broadcast and not multicast            skip the shouting
net 192.168.1.0/24 and not host 192.168.1.1   the LAN minus the router
```

Notice how much poorer this is than
chapter 6: no "DNS queries mentioning
update", no "retransmissions", no
"packets the HTTP dissector claimed".
BPF cannot know those things yet.
`port 53` is not "DNS" — it is any
packet wearing that number, and DNS on
an unusual port sails past it.

## When to filter at capture, and when not to

**Filter at capture when** the capture
must run long or the link is busy: an
hour of a household's whole traffic is
millions of packets, and a pcapng grows
by gigabytes. A capture filter aimed at
one sick machine — `host 192.168.1.20` —
turns a flood into a file you can
actually open.

**Filter at display when** the capture
is short, or you do not yet know what
you are looking for — which is most of
the time, early in a diagnosis. The
cost of recording everything for five
minutes is a bigger file; the cost of
recording the wrong thing is the whole
investigation.

A middle path exists and is often the
right one: capture broadly but bounded —
dumpcap's **ring buffer** (chapter 10)
keeps, say, the last ten 100 MB files
rolling on disk, so a capture can wait
all day for an intermittent fault
without eating the disk. When the fault
happens, you stop the capture and the
recent past is already saved.

## Two traps

**The filter that eats the evidence.**
"Ignore the noise" filters written at
capture time (`not arp and not dns`)
have a way of discarding exactly the
packets that would have explained the
fault — a DNS failure *is* DNS packets.
Noise filtered at display costs nothing.
Noise filtered at capture is a bet.

**VLAN and tunnel offsets.** BPF matches
bytes at expected places. Add a VLAN tag
and every expected place moves by four
bytes; libpcap has the `vlan` keyword to
say so, and forgetting it makes filters
silently match nothing on tagged
networks. If a filter that "can't be
wrong" catches nothing at all on a
managed network, this is the first
suspect (chapter 15 returns to it).

Check: on a loopback capture,
`-f "icmp"` records only the pings, and
`capinfos` (chapter 10) on the resulting
file shows a packet count to match —
proof the filter acted at capture, not
at reading, because the other packets
were never in the file to count.
