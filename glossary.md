# Glossary

**BPF** (Berkeley Packet Filter) — The
small, fast filter language capture
filters are written in; compiled by
libpcap and applied before packets are
recorded (chapter 7).

**Capture filter** — A BPF expression
that decides which packets are recorded
at all. Applied by the capture library;
what it rejects is gone for good
(chapter 7).

**capinfos** — The file tool that
reports on a capture file: packet count,
duration, sizes, hashes, and the
application that made it (chapter 10).

**Checksum offloading** — Letting the
network card fill in packet checksums.
Captures taken on the sending machine
see the packets before that happens and
cry "bad checksum" wrongly (chapters
11 and 15).

**Conversation** — All the packets
between one pair of endpoints (for TCP,
one connection). The unit of the
conversations table and of *follow
stream* (chapter 8).

**Dissector** — One of Wireshark's
protocol translators. Dissectors chain:
each reads its layer's header and hands
the rest to the next (chapter 2).

**Display filter** — Wireshark's own
filter language, applied to packets
already captured. Hides; never deletes
(chapter 6).

**dumpcap** — The small privileged
program that actually grabs packets and
writes them to disk, so the big programs
need no special rights (chapters 2, 3).

**editcap** — The file tool that copies
a capture selectively: by packet range,
by time window, or without duplicates
(chapter 10).

**Endpoint** — One address (with a port,
where the protocol has them) taking part
in traffic; endpoints statistics total
each one's packets and bytes (chapter 8).

**Follow stream** — The view that
reassembles one conversation's data in
order, as the two programs exchanged it
(chapter 8).

**Frame** — One captured unit as stored
in a file, including Wireshark's own
book-keeping branch in the dissection
tree. Used loosely, "one packet"
(chapter 5).

**libpcap** — The capture library on
Linux and macOS: asks the system for
copies of frames and applies capture
filters. tcpdump uses the same library
(chapters 2, 13).

**mergecap** — The file tool that joins
several captures into one, interleaved
by timestamp (chapter 10).

**Monitor mode** — A Wi-Fi adapter mode
that captures all frames in the air, not
just its own network's. Support varies
sharply by adapter and system
(chapter 14).

**Npcap** — The capture library for
Windows, installed with Wireshark; the
Npcap service is what lets ordinary
users capture (chapter 3).

**Packet** — One addressed envelope of
data at the network layer, and, loosely,
the frame that carried it (chapter 1).

**pcap** — The elder capture file
format: a plain stream of frames with
timestamps, read by everything
(chapter 2).

**pcapng** — The current capture file
format Wireshark writes by default:
multiple interfaces per file, comments
on packets, notes about the capturing
machine (chapter 2).

**Profile** — A named bundle of your
filters, columns, colouring rules and
preferences. The window shows the
current one in its status bar
(chapters 2, 11).

**Protocol hierarchy** — The table of
contents of a capture: every protocol
found, nested, with frame and byte
counts (chapter 8).

**Promiscuous mode** — An interface
accepting frames addressed to other
machines as well as its own. Default
for captures; on a switched network
there is little extra for it to accept
(chapters 1, 14).

**Ring buffer** — A rolling set of
capture files of fixed total size, for
unattended captures that wait for an
intermittent fault (chapters 9, 10).

**Snap length** — A capture-time limit
on how much of each frame is stored.
Shorter files; dissections that stop
early (chapter 5).

**SPAN / mirror port** — A managed
switch feature that copies other ports'
traffic to one port, so a capture there
sees conversations it is not part of
(chapter 14).

**Stream index** — The number tshark
gives each conversation in a file
(`tcp.stream`), used by *follow* and by
filters (chapter 8).

**Tap** — A small hardware device on a
cable that copies the traffic passing
through it to a listener port
(chapter 14).

**text2pcap** — The file tool that
rebuilds a capture file from a hex dump
held in a text file (chapter 10).

**tshark** — The dissection engine with
a command-line face: capture, read,
filter, extract fields, print statistics
(chapter 9).
