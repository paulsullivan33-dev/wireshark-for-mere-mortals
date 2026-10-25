# The Neighbours

Wireshark's street is a busy one. These
are the tools it lives among — two of
them have full guides in this series —
with the honest line between what each
does and what Wireshark does.

## tcpdump

The elder of the street, and the one
Wireshark shares plumbing with: both use
libpcap, and a capture filter
(chapter 7) means exactly the same thing
to each. tcpdump is the command-line
capturer found on practically every Unix
machine ever shipped, small enough for
routers and rescue shells where
Wireshark's crew will never fit. Its
reading abilities are a window's poor
cousin — a few lines of decode per
packet rather than the full tree — so
the common working pattern is *capture
with tcpdump on the machine that can,
read with Wireshark on the machine with
a screen*:

```bash
tcpdump -i eth0 -w router.pcapng host 192.168.1.20
```

If this guide's tshark chapters feel
comfortable, tcpdump's flags will take
an afternoon to learn; the mental model
is identical, because underneath, it is
the same machine. The series' tcpdump
guide covers it in full.

## Nmap

The neighbour who knocks. Nmap *sends*
packets — probes — to discover hosts,
open ports and services, where Wireshark
only listens to what passes. They are
natural partners read in that order:
Nmap tells you what a machine offers;
Wireshark shows you the offering being
used, abused or fumbled. A port scan
captured in Wireshark is also the
kindest way to learn both tools at once:
the probes and the answers, side by
side. The series' Nmap guide is the
place for the knocking itself.

## Terminal-only analysers: Zeek and friends

**Zeek** (once named Bro) watches a
network the way Wireshark does, but
instead of packets it writes *stories*:
log files of connections, DNS lookups,
HTTP requests, files transferred, each
already summarised. Networks too fast or
too big for packet files live here.
**Suricata** and **Snort** watch for
*known-bad* shapes in traffic and raise
alerts — the burglar-alarm end of the
street, where Wireshark is the security
camera you rewind by hand. Neither
replaces the packet-level truth; both
tell you which five minutes of it to
rewind.

## mitmproxy

An HTTP(S) proxy you place in the middle
on purpose: your test device points at
it, and it shows — and can rewrite —
requests and responses as readable HTTP
instead of a packet stream. For
debugging an app's API chatter it is
often the faster tool, because it starts
at the layer you care about. What it
cannot do is anything that is not HTTP
through the proxy: the DNS that failed
beforehand, the other device's traffic,
the raw TCP behaviour — that is
Wireshark's ground again.

## When the neighbour is the better tool

| Need | Reach for |
|---|---|
| Capture on a router or rescue shell | tcpdump |
| What is running on that machine at all | Nmap |
| A month of "what happened on this network" | Zeek |
| Alerts when known-bad traffic appears | Suricata |
| Read an app's API calls as text | mitmproxy |
| What exactly did these two machines say | **Wireshark** |

The last row is the home turf, and it is
narrower than the others on purpose:
nothing else on the street shows the
packets themselves, all of them, in
order, decoded to the last flag bit.
That is the whole job — chapter 14 is
about what even that cannot do.
