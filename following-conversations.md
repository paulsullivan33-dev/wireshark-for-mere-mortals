# Following Conversations

A packet list is a crowd shouting single
words. This chapter is about seating two
of the talkers at a table by themselves:
**streams** (one conversation,
reassembled) and the **tables** that
summarise all of them at once.

## Follow the stream

In the window: right-click any packet of
a conversation → *Follow → TCP Stream*.
The window narrows to that conversation
and opens its contents as a document —
one side's words in one colour, the
other's in another, with the bytes
neither side can deny. On the command
line, the same view is:

```bash
tshark -r lab.pcapng -q -z follow,tcp,ascii,0
```

(the `0` is the **stream index**, the
number the dissection tree showed in
chapter 5). The lab's stream 0 reads:

```
Node 0: 127.0.0.1:33368        Node 1: 127.0.0.1:8080
GET / HTTP/1.1
Host: 127.0.0.1:8080
User-Agent: curl/8.5.0

HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.12.3
Content-type: text/html
Content-Length: 181

<!DOCTYPE html>
<html><head><title>Loopback Tea Room</title></head>
…
```

Three things follow-the-stream teaches
that the packet list cannot. **The
application protocol becomes prose**:
headers and body in order, gaps and
re-sends ironed out — this is where a
mystery protocol stops being mysterious.
**Order is repaired**: the stream is
reassembled from segments by sequence
number, so the words appear as the
receiving program saw them, not as the
wire shuffled them. **The endings are
visible**: scroll to the bottom and the
conversation's last words — a polite
FIN exchange or a RST, mid-sentence —
are the first place chapter 12 sends
you when an app "just drops".

There are Follow variants for UDP,
HTTP (which shows each request and
response as a unit across redirects),
TLS (the handshake in detail; payload
only with keys, chapter 14), and any
protocol with a stream notion. The
filter the window applies when you
follow — `tcp.stream eq 0` — is a
display filter like any other
(chapter 6), and worth stealing for
command-line use.

## The tables: conversations and endpoints

Statistics → **Conversations** lists
every conversation the file holds:

```
127.0.0.1:33368 <-> 127.0.0.1:8080   12 frames 1252 bytes  start 1.0609  dur 0.0107
127.0.0.1:33370 <-> 127.0.0.1:8080   12 frames 1215 bytes  start 1.1782  dur 0.0101
127.0.0.1:33378 <-> 127.0.0.1:8080   12 frames 1414 bytes  start 1.2266  dur 0.0120
127.0.0.1:58002 <-> 127.0.0.1:9       2 frames  128 bytes  start 1.2987  dur 0.0000
```

(from `-q -z conv,tcp`, transcript t06.)
The lab file resolves into exactly four
stories: three web fetches and the
refusal. On a real capture this table is
the map of the territory — sorted by
bytes it answers "who is doing all
that?", sorted by duration it finds the
connections that linger, and clicking a
row applies that conversation as a
filter. **Endpoints** is the same table
rolled up per address: every host, its
packet and byte totals. (`-q -z
endpoints,ipv4` on the command line.)

One loopback honesty note: the lab's
endpoint table claims 127.0.0.1 took
part in 92 packets — the file's 46,
counted once as sender and once as
receiver, because on loopback every
endpoint is the same machine. On a real
interface the senders and receivers are
different rows, and the table reads the
way it promises.

## The protocol hierarchy

Statistics → **Protocol Hierarchy** is
the table of contents of the file:

```
eth    frames:46 bytes:4793
  ip   frames:46 bytes:4793
    icmp  frames:8  bytes:784
    tcp   frames:38 bytes:4009
      http frames:6 bytes:1296
```

(`-q -z io,phs`.) Read it before
anything else on an unfamiliar capture:
it says what the file is *made of*, in
the dissectors' opinion, and half of all
"the network is weird" captures confess
in this table — an ocean of one chatty
protocol nobody expected, or the total
absence of the one you came for.
