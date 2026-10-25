# Wireshark for Mere Mortals

Every program on a network tells a story
about what it sent. The server's log says
the page was served; the browser says it
waited; both are remembering, politely.
**Wireshark** is the tool that was
*watching*: it records the packets
themselves — every request, answer,
re-send and refusal — and decodes them
into a form a patient human can read.
This guide explains what a packet
analyser is in those terms, how to
install Wireshark and take your first
capture, how to read one packet layer by
layer, how to filter a flood down to the
five packets that matter, and how to
follow a whole conversation — with the
limits stated plainly, including the big
one: most modern traffic is encrypted,
and no tool reads what mathematics has
sealed.

The lab behind this guide ran the real
thing: **Wireshark 4.2.2**, the version
in Ubuntu 24.04's repositories —
specifically its command-line half,
**tshark**, with dumpcap and the file
tools — installed with apt onto a small
Linux machine. Against a throwaway web
server on the machine's own loopback
interface, the lab caused traffic on
purpose and captured it: pings, three
complete web fetches (one of them an
honest 404), and one connection refused
on the spot. That one 46-packet file was
then exercised end to end: dissected
frame by frame, filtered eight ways,
its first conversation followed and
reassembled, its statistics printed,
its fields extracted as a table and as
JSON, and the file itself reported on,
sliced by range, sliced by time, merged
with another slice, and even rebuilt
from a hex dump. A second capture, taken
under a capture filter, proved the
filter's power by what it *didn't*
record. Even the install had to be done
honestly: the unattended package setup
skipped the capture-permissions step, so
the lab applied it by hand and kept the
receipts.

What the lab could not do, it did not
fake. The machine had **no screen**, so
the famous three-pane window — the way
most people meet Wireshark — is taught
from the project's documentation and
labelled as such wherever it appears;
every claim about packets, filters and
files was tested with tshark on real
captures instead, which is possible
because the window and tshark are one
engine wearing two faces. For the same
reason, captures here were taken on
**loopback**, the one interface every
reader's machine shares: chapter 4
names the ways loopback simplifies the
picture (no real hardware addresses, one
address at both ends), and the chapters
on real networks — switched ports,
Wi-Fi, other people's traffic — are
marked as documentation wherever the
lab could not sit in that seat.
`criteria.md` keeps the full ledger of
what was run and what was only read.

## What this guide covers

- [**What Wireshark is**](what-wireshark-is.md) —
  the mental model: packets, frames, the
  one-machine rule, and what a packet
  analyser is not
- [**The moving parts**](the-moving-parts.md) —
  dumpcap, the capture library, the
  dissectors, tshark, the window, and
  the file tools: which part holds what
- [**Installing Wireshark**](installing-wireshark.md) —
  the install on Windows, macOS and
  Linux, and the permission-to-capture
  setup that decides whether it works
- [**Your first capture**](your-first-capture.md) —
  a loopback capture of traffic you
  caused yourself, and how to read what
  you caught
- [**Reading a packet**](reading-a-packet.md) —
  one real frame taken apart: frame,
  Ethernet, IP, TCP, HTTP, and what each
  layer's fields are really saying
- [**Display filters**](display-filters.md) —
  the filter language for packets
  already captured: fields, comparisons,
  and the recipes worth memorising
- [**Capture filters**](capture-filters.md) —
  BPF, the smaller language that decides
  what gets recorded at all, and when
  that bet is worth placing
- [**Following conversations**](following-conversations.md) —
  follow-stream reassembly, and the
  conversation, endpoint and hierarchy
  tables that map a whole file
- [**TShark at the command line**](tshark.md) —
  fields, JSON, statistics with `-z`,
  and dumpcap's ring buffer for captures
  nobody watches
- [**Working with capture files**](capture-files.md) —
  capinfos, editcap, mergecap and
  text2pcap: reporting, slicing, joining
  and resurrecting capture files
- [**Names and colour**](names-and-colour.md) —
  name resolution and its traps,
  colouring rules, marking and comments,
  and profiles that bundle it all
- [**Recipes**](recipes.md) — five home
  network jobs: the unknown device, the
  slow page, lying DNS, the restarting
  connection, the noisy neighbour
- [**The neighbours**](the-neighbours.md) —
  tcpdump, Nmap, Zeek, Suricata and
  mitmproxy: what each does that
  Wireshark doesn't, and vice versa
- [**Limits**](limits.md) — one seat on
  a switched network, the encryption
  wall, firehose arithmetic, and the
  legal edge, stated plainly
- [**Troubleshooting**](troubleshooting.md) —
  permission denied, the empty capture,
  the doubled capture, and alarms that
  are not real
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Decrypting other people's traffic —
there is no such chapter in any honest
book, and chapter 14 explains the three
legitimate doors (metadata, your own
endpoints' keys, a seat inside the
endpoint) and why there is no fourth.
Wireless monitor-mode work in depth,
evidence handling for legal proceedings,
and analysing captures at backbone scale
are likewise pointed at, not taught.
Writing your own dissector in Lua is a
fine afternoon and a different guide.

## How to use this guide

Read chapter 1 for the model and
chapter 2 before installing anything —
knowing which part of the crew holds
what turns later confusion into a named
problem. Take chapter 4's capture in the
same sitting as the install, while the
permissions are fresh; manufactured
traffic you caused yourself is the
kindest teacher this subject has.
Chapter 6 is the one to live in
afterwards: most real questions end at a
display filter. Chapter 12 assumes you
own the network you are capturing, and
chapter 14 is not the optional chapter —
read it before pointing Wireshark at
anything that carries other people. The
lab's transcripts live in
`wireshark-lab/transcripts/`, the lab
capture itself beside them, and its
notes are `examples/lab-notes.txt`.
