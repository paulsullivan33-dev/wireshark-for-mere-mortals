# What Wireshark Is

Wireshark is a **packet analyser**. It
watches the traffic crossing a network
connection — a cable, a Wi-Fi radio, the
loopback inside one machine — and records
each packet: the headers that route it,
the protocol it speaks, and the data it
carries, all decoded into something a
human can read. Where a browser shows you
the finished page and a server log shows
you the request it remembers, Wireshark
shows you the conversation itself, packet
by packet, in the order it happened.

That makes it the tool you reach for when
the question is not "what did the program
say happened" but "what actually went over
the wire":

- A device is slow, and you want to know
  whether the network or the program is
  at fault.
- Two machines disagree about what was
  sent, and both logs look innocent.
- Something on your network is talking to
  a stranger, and you want its name.
- You are learning how a protocol works
  and want to watch it instead of reading
  about it.

## Packets, frames, and the one machine rule

A **packet** is one addressed envelope of
data. On a cable it travels inside a
**frame** — the envelope's envelope — and
the two words are often used loosely for
the same thing. Wireshark lists frames;
inside each one it dissects (takes apart)
the layers: the link layer that moved it
across one hop, the network layer (usually
IP) that addresses it across the world,
the transport layer (usually TCP or UDP)
that carries it between two programs, and
the application protocol on top — HTTP,
DNS, TLS and thousands more. Chapter 5
reads one packet layer by layer.

One rule governs everything in this
guide: **Wireshark can only record
traffic that physically reaches the
machine it runs on.** On your own
computer that means your traffic, plus
broadcasts addressed to everyone. It does
not mean your neighbour's traffic, and on
a modern switched network it does not
even mean the other ports of your own
switch — the switch delivers each frame
to its destination port only. Capturing
*other* machines' traffic needs their
cooperation: a mirror (SPAN) port on a
managed switch, a network tap, or running
the capture on one of the machines
involved. Chapter 14 is honest about
this and the rest of the limits.

## Two halves: the window and the engine

Most people meet Wireshark as a desktop
window with three panes — a list of
packets, a tree of decoded layers, and
the raw bytes. Underneath it sits the
same engine in command-line form:
**tshark**, which captures and analyses
with no window at all, and **dumpcap**,
the small privileged program that does
the actual grabbing. Everything the
window can show, tshark can print;
everything in this guide is taught both
ways where the two differ. The small
print matters on a home network: if you
can capture it on a router, a Raspberry
Pi or a server with no screen, tshark is
how you will read it later — very likely
by copying the file to a machine with a
screen and opening it in the window.

## What it is not

- **Not a security scanner.** Wireshark
  finds nothing and attacks nothing; it
  watches. (Nmap, chapter 13, is the
  neighbour that goes knocking.)
- **Not a log.** It records only while it
  is running, and a busy link produces
  millions of packets an hour. Captures
  are evidence you gather on purpose,
  not a history the machine keeps.
- **Not an X-ray for encryption.** A
  modern capture is mostly TLS, and
  Wireshark shows the handshake, the
  names and the sizes — not the contents —
  unless you hand it the session keys.
  Chapter 14 explains what can and
  cannot be done about that.
- **Not proof by itself.** A capture
  shows what one machine saw, from its
  seat on the network. Frames it never
  received are not in it, and a busy
  capture can silently drop some
  (chapter 15).

Check: after installing (chapter 3),
`tshark --version` prints the version
the whole guide was tested against, and
`tshark -D` lists the interfaces this
machine could capture from. If both
answer, you have the engine; the window
is the same machinery wearing a face.
