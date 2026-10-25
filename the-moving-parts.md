# The Moving Parts

Wireshark looks like one program and is
really a small crew. Knowing which part
holds what turns most confusion into a
named problem, so here is the crew before
you install anything.

## The pipeline

Every capture walks the same road:

```
interface → dumpcap → capture file → dissection → you
```

1. The **capture library** asks the
   operating system for copies of frames
   as they pass a network interface.
2. **dumpcap** writes those frames, as
   fast as it can, into a capture file.
3. The **dissectors** — Wireshark's
   library of protocol translators —
   take a saved frame apart and explain
   every field.
4. You read the result in the **window**
   or through **tshark**.

Capture and reading are separate acts,
and that separation is deliberate: the
part that must never fall behind (writing
frames at wire speed) is small and does
no thinking, while the thinking part can
take its time over a finished file.

## The parts, one by one

**The capture library.** On Linux and
macOS this is **libpcap**, the same
library tcpdump uses (chapter 13). On
Windows it is **Npcap**, installed
alongside Wireshark. It is the only
layer that talks to the network driver,
and it is where capture filters
(chapter 7) are compiled and applied.

**dumpcap.** The only part of the crew
that needs special rights, because
opening a network interface for raw
capture is a privileged act. It is kept
deliberately tiny for that reason: it
grabs frames, writes them to disk (or a
ring of files), and does nothing else.
The big programs — window and tshark —
run unprivileged and hand the grabbing
to dumpcap. Chapter 3 covers how the
rights are granted on each system, and
why you should not simply run the whole
of Wireshark as root or administrator.

**The dissectors.** Several thousand
protocol translators, one per protocol,
organised as a chain: the Ethernet
dissector reads its header, sees "IP
inside", and hands over; IP sees "TCP"
and hands over; TCP sees port 443 and
hands over to TLS. This chain is why a
single frame can be unfolded layer after
layer in chapter 5's tree, and why a
packet sometimes stops being decoded
halfway — no dissector claimed the next
layer, or one disagreed about what it
saw ("malformed" packets, chapter 15).

**tshark.** The whole dissection engine
with a command-line face: capture live,
read files, filter, extract fields,
print statistics. Chapter 9 is its
chapter. Scripts and small headless
machines live here.

**The window.** The desktop program:
packet list, dissection tree, raw bytes,
plus the analysis furniture — stream
following, conversation tables, graphs,
expert information. Everything it shows
comes from the same dissectors tshark
uses, so a file means the same thing in
both. This guide's lab had no screen, so
the window is taught from the
documentation and labelled as such
wherever it appears; every claim about
*packets* in this guide was tested with
tshark on real captures.

**The file tools.** A supporting cast of
small utilities, each doing one job on
capture files:

| Tool | Job |
|---|---|
| `capinfos` | Report on a file: how many packets, how long, how big |
| `editcap` | Trim, split, slice by time, remove duplicates |
| `mergecap` | Join several captures into one, in time order |
| `text2pcap` | Turn a hex dump in a text file back into a capture |
| `dumpcap` | Capture to file, with ring buffers for long runs |

Chapter 10 puts the first three to work.

## The file formats

Two formats matter. **pcap** is the
elder: a simple stream of frames with
timestamps, understood by everything.
**pcapng** is what Wireshark writes by
default today: it can hold several
interfaces in one file, comments on
individual packets, and notes about the
machine that captured it. Both open in
either tool; pcap is the safer choice
when a file must travel to other
software. A file name ending in
`.pcapng` or `.pcap` tells you the
format; `capinfos` (chapter 10) tells
you for certain.

## Where your settings live

Wireshark keeps **profiles** — a named
bundle of your filters, colouring rules
(chapter 11), columns and preferences.
The factory profile is called *Default*.
Profiles are how the same program can be
a DNS-debugging desk for one job and a
VoIP desk for another without disturbing
either. tshark honours profiles too
(`-C <name>`), which is one of several
reasons a capture analysed on the
command line can look different from the
same file in a colleague's window: same
dissectors, different furniture.

Check: `tshark -D` lists interfaces,
`tshark --version` names the version
and the library versions it was built
with, and `capinfos` on any capture
file you already own names its format.
Three commands, three parts of the crew
reporting for duty.
