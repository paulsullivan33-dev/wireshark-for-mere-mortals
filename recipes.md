# Recipes

Five jobs this guide's readers actually
bring to a capture. Each assumes a home
network you own, a capture taken on or
near the machine in question (chapter 14
explains why the seat matters), and the
filters from chapters 6 and 7.

## 1. What is this device talking to?

*Symptom: an unfamiliar device on the
LAN, or a familiar one behaving oddly.*

Capture five minutes of that device
only: capture filter `host 192.168.1.20`.
Then ask the file three questions:

```bash
tshark -r cap.pcapng -q -z endpoints,ipv4
tshark -r cap.pcapng -q -z conv,udp
tshark -r cap.pcapng -Y "dns" -T fields -e dns.qry.name | sort -u
```

The DNS names are the plainest answer —
a device asking for `telemetry.…` and
`pool.ntp.org` is telling two different
stories. Endpoints with no DNS (an IP
talked to but never looked up) deserve a
second look: hard-coded addresses are
where the interesting behaviour lives.

## 2. The slow web page

*Symptom: one site crawls; the rest of
the network is fine.*

Capture on the suffering machine while
loading the page once, then walk the
ladder:

1. `dns` — how long between query and
   answer? A second of silence here is
   the whole complaint, wearing a
   network costume.
2. `tcp.flags.syn == 1` — how long from
   SYN to SYN-ACK? That is the raw
   round trip to the server.
3. `tls.handshake` — how many round
   trips before data? Each is the delay
   from step 2, paid again.
4. `tcp.analysis.retransmission` — is
   anyone re-sending? Loss on the path
   multiplies everything above.

Most "slow site" captures are decided
by step 1 or 4, and both answers are
visible without decrypting a byte.

## 3. DNS that lies or limps

*Symptom: names fail, resolve slowly, or
resolve to the wrong place.*

Display filter `dns` on the client.
What healthy looks like: each query
answered in milliseconds, flags showing
no error. The failure shapes to filter
for:

```text
dns.flags.rcode != 0          NXDOMAIN and friends — the name truly failed
dns.time > 0.5                the reply took over half a second
dns.count.answers == 0 and dns.flags.response == 1
                              "no error, and also no answer" — upstream oddity
```

And the quiet classic: the client asks
**both** its IPv4 and IPv6 resolvers, one
of which is dead — half the queries in
the file are simply never answered.
`dns.qry.name` sorted beside `ip.dst`
makes the dead resolver obvious.

## 4. The connection that keeps restarting

*Symptom: an app reconnects, stalls, or
drops at the same point in a task.*

Follow the stream (chapter 8) and read
the ending — every TCP conversation
announces how it died:

- **RST from one side**: that side gave
  up abruptly; the sender of the reset is
  the first suspect, and the packets just
  before it are why.
- **Retransmissions, then silence**: the
  path ate the packets; look for the same
  shape at other times of day (Wi-Fi
  interference keeps office hours).
- **Zero window**: the *receiving*
  program stopped reading. Not a network
  fault at all — a busy or wedged app,
  and the capture that proves it saves a
  long argument with the network.

`tcp.analysis.retransmission or
tcp.flags.reset == 1` is the filter
that gets you to those endings fastest.

## 5. One noisy neighbour

*Symptom: the LAN feels busy; nothing
specific is broken.*

Statistics → Endpoints (or
`tshark -q -z endpoints,ipv4`), sorted
by packets. The top talker on a home
network at idle should be something you
can name. Then filter to it
(`ip.addr == …`) and ask chapter 6's
question — what protocol, what rhythm?
A device emitting a broadcast every
second, forever, is usually a discovery
protocol having a bad day; the packet
itself names the protocol, and the
protocol names the setting to go and
turn off.

## A note on all five

Each recipe names a question the packets
can answer and stops there — the fix
lives in a router setting, a dead
resolver, a wedged app, and finding it
is yours. That is the honest shape of
this tool at home: Wireshark rarely
repairs anything, but it reliably ends
arguments about whose layer it is.
