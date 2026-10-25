# Display Filters

A display filter decides which of the
packets **already captured** you are
looking at. Nothing is thrown away: the
packets leave the screen, not the file,
and clearing the filter brings them all
back. That makes display filters the
safe place to experiment — every recipe
below can be tried, undone, and retried
on the same capture, including this
guide's lab file.

In the window, the filter goes in the
bar under the toolbar (it glows red when
the filter cannot be understood, green
when it can). In tshark it follows `-Y`:

```bash
tshark -r lab.pcapng -Y "http"
```

## The shape of the language

A display filter names **fields** — the
same fields the dissection tree shows —
and compares them with values:

| Piece | Example | Means |
|---|---|---|
| Protocol | `dns` | Any packet DNS was decoded in |
| Field | `ip.addr` | A field that exists in the packet |
| Equality | `ip.addr == 192.168.1.20` | Field equals value |
| Ports | `tcp.port == 443` | Source *or* destination port |
| Contains | `http.host contains "example"` | Text appears in the field |
| Set | `tcp.port in {80 443 8080}` | Field equals any of these |
| Logic | `dns and not arp` | Combine with `and`, `or`, `not` |

Three habits separate comfortable
filtering from fighting:

**`ip.addr` matches either end.**
`ip.addr == 192.168.1.20` shows packets
to *and* from that address; `ip.src` and
`ip.dst` exist for when direction
matters. Nearly every field with two
ends behaves this way — `tcp.port` is
both ports, `tcp.srcport` is one.

**A bare field name tests existence.**
`tls.handshake` shows packets *having*
that field. `http` alone means "packets
the HTTP dissector claimed", which is
not quite "packets on port 80" — a web
server on port 9000 is still HTTP to
the dissector, and a game on port 80 is
not.

**Comparisons are per-field, per-layer.**
A packet carrying two IP layers (a
tunnel) can match a filter through
either one. The filter engine asks "does
*any* instance of this field satisfy
this?" — convenient, and the root of
most "why did that match?" surprises.

## Recipes worth memorising

```text
ip.addr == 192.168.1.20                      one machine, both directions
ip.addr == 192.168.1.0/24                    a whole subnet (CIDR works)
tcp.port == 8080                             one service, either end
dns.qry.name contains "update"               lookups mentioning a word
http.request                                 requests only, not replies
tcp.flags.syn == 1 and tcp.flags.ack == 0    connection starts
tcp.analysis.retransmission                  TCP saying "again, please"
frame.len > 1000                             the big ones
not (arp or dns or tcp.port == 443)          hush the usual noise
```

`tcp.analysis.*` deserves a paragraph of
its own. Wireshark's TCP dissector
remembers every conversation as it reads,
and attaches its conclusions to the
packets: `.retransmission`,
`.duplicate_ack`, `.zero_window`,
`.out_of_order`. Filtering on those is
the fastest known route from "the
network feels slow" to a specific packet
doing a specific wrong thing, and
chapter 12 leans on it heavily.

## Slicing and membership tricks

Two operators earn their keep on long
captures. The **slice** operator cuts a
field down to bytes: `eth[0] & 1` tests
the multicast bit of a MAC address; it
is how you filter on things no dissector
named. **Membership** with `in` takes a
set, and the set can hold ranges:
`tcp.port in {443 8000..8100}`. And
`matches` applies a regular expression:
`http.host matches ".*\\.local$"` for
names ending in `.local`.

## Building filters without tears

- **Right-click is the author.** In the
  window, right-clicking any field in the
  dissection tree offers *Apply as
  Filter*, which writes the exact field
  name for you. Guessing field names is
  optional and error-prone; the tree is
  the dictionary.
- **The filter is also a column maker.**
  *Apply as Column* turns a field into a
  packet-list column — the trick behind
  chapter 11's tuned layouts.
- **Bookmarks are built in.** The filter
  bar's bookmark button saves filters
  with names; every reader should save
  the four or five their own network
  keeps demanding.

Check: on the lab capture,
`tshark -r lab.pcapng -Y "icmp"` lists
only the 8 ping packets, `-Y "http"`
the 6 frames of the three web fetches,
and `-Y "not tcp and not icmp"` comes
back empty — three filters agreeing
that the file holds pings and TCP
conversations and nothing else.
Chapter 9 shows the same filters doing
statistics and field extraction.
