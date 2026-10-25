# Reading a Packet

One packet, read properly, teaches the
whole interface. This chapter takes
frame 8 of the lab capture — the
`GET /` request from chapter 4 — apart
layer by layer, exactly as the
dissection tree shows it. In the window
the same tree is the middle pane, with
the raw bytes below it lighting up as
you click each line; in tshark the tree
is what `-V` prints (transcript t04).

## The packet list, first

A line in the list is a summary written
by the dissectors:

```
8 1.061011269  127.0.0.1 → 127.0.0.1  HTTP 143 GET / HTTP/1.1
```

Number, time since the capture's first
frame, source, destination, protocol of
the *topmost* layer decoded, length,
and an info column in that protocol's
own words. Everything in the tree below
exists to justify this one line — when
a summary surprises you, the tree is
where it confesses.

## Frame: the part Wireshark adds

```
Frame 8: 143 bytes on wire (1144 bits), 143 bytes captured
    [Protocols in frame: eth:ethertype:ip:tcp:http]
```

The first branch of the tree is not in
the packet at all: it is Wireshark's own
notebook entry *about* the packet — when
it arrived, how long it is, and the
whole protocol stack in one line:
`eth:ethertype:ip:tcp:http`. When the
two lengths differ ("on wire" longer
than "captured"), the capture was taken
with a size limit (a *snap length*);
chapter 15 meets that again.

## Ethernet: one hop's envelope

```
Ethernet II, Src: 00:00:00_00:00:00, Dst: 00:00:00_00:00:00
    Type: IPv4 (0x0800)
```

Source and destination **MAC
addresses** — the names of the two
network interfaces at either end of
*this hop*, which on a real network are
rarely the journey's start and end (your
packet to a website wears your router's
MAC, not the website's). The `Type`
field is the hand-over note: `0x0800`
means "IPv4 inside". On the lab's
loopback the addresses are all zeros —
there is no hardware on a pretend
interface, so the envelope is a stub.
On Ethernet this branch also carries
VLAN tags when present (chapter 7).

## IP: the address that crosses the world

```
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
    Header Length: 20 bytes
    Total Length: 129
    Flags: 0x2, Don't fragment
    Time to Live: 64
    Protocol: TCP (6)
```

The fields worth a newcomer's attention
are few. **Total Length** is the whole
IP packet, headers included — compare
it with the frame's 143: the 14-byte
difference is the Ethernet header
outside it. **Time to Live** counts the
routers the packet may still cross; it
arrives smaller than it left, and the
gap roughly counts the hops. **Protocol**
is this layer's hand-over note: 6 means
TCP inside. (An IPv6 branch reads the
same way with different furniture: no
header checksum, a *Hop Limit* instead
of TTL, and addresses long enough to
need the abbreviation `::`.)

## TCP: the conversation machinery

```
Transmission Control Protocol, Src Port: 33368, Dst Port: 8080
    Sequence Number: 1    (relative sequence number)
    Acknowledgment Number: 1    (relative ack number)
    Flags: 0x018 (PSH, ACK)
    Window: 64
    [Calculated window size: 65536]
    [Stream index: 0]
    [Conversation completeness: Incomplete, ESTABLISHED (7)]
```

Ports name the two *programs*: 8080 is
the little web server; 33368 is the
ephemeral port the system lent to curl
for this one conversation. **Sequence**
and **acknowledgment** numbers are the
book-keeping that lets TCP notice loss
and reassembly — you rarely read them
directly; chapter 6's `tcp.analysis`
fields are the dissectors reading them
for you. Wireshark shows them *relative*
to the conversation's start (the raw
values, which are random, sit beside
them) — one of many small kindnesses the
annotations perform. The **flags** are
the traffic signals: SYN starts, FIN
ends politely, RST ends abruptly, ACK
acknowledges, PSH says "deliver this
now". The **window** lines show another
annotation at work: the packet carries
64, and the dissector multiplies by the
scaling factor negotiated at the
handshake (1024) to report the true
65536. And the bracketed completeness
verdict is honest in a way worth
noticing: at frame 8 the conversation
is *still running*, so it reads
"Incomplete, ESTABLISHED" — the
annotation describes what the capture
has shown *so far*, not how the story
ends.

## HTTP: the payload, dressed

```
Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
    Host: 127.0.0.1:8080\r\n
    User-Agent: curl/8.5.0\r\n
```

At the top of the stack the headers
become human text, because HTTP is a
text protocol: the request line, the
headers, and (in the reply, frames
10–12) the body itself — the lab page's
HTML, carried in a TCP segment and
reassembled by the dissector before
being shown whole. With TLS instead of
HTTP, this branch would show a handshake
and then stop at "Application Data":
the layers below stay fully readable,
the top one goes private (chapter 14).

## The two-packet story at the end of the file

Frames 41 and 42 repay a look before
leaving this chapter. Frame 41 is a SYN
to port 9, where no program listens;
frame 42 is the entire reply:

```
42  TCP 54  9 → 58002 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
```

One packet, no handshake, conversation
over: that is what "connection refused"
looks like on the wire, and recognising
the shape — SYN, RST, silence — is half
of chapter 12's diagnostics already in
your pocket.

Check: `tshark -r lab.pcapng -V` on
frame 1 shows the same tree in miniature
— frame, Ethernet, IP, and then ICMP
with its type (8, a request; frame 2
answers with type 0) and the sequence
number the two directions share.
