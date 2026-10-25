# Working with Capture Files

Captures outlive the questions that
caused them. They get emailed to
colleagues, attached to bug reports,
reopened next year. This chapter is the
care and feeding of the files
themselves — all of it done in the lab
on the real capture (transcript t08).

## capinfos: ask the file who it is

```text
$ capinfos lab.pcapng
File type:            Wireshark/... - pcapng
Number of packets:    46
Capture duration:     2.369972170 seconds
First packet time:    2026-10-25 00:27:13.693198476 UTC
Data size:            4793 bytes
Capture application:  Dumpcap (Wireshark) 4.2.2
```

Forty-six packets, two and a bit
seconds, one interface (`lo`, listed
further down). capinfos also prints
checksum hashes of the file — the
receipt that the file your colleague
opens is the file you sent — and works
on hundreds of files at once, which
makes it the census tool for a drawer
full of old captures: `capinfos -c
*.pcapng` prints just the packet counts.

## editcap: the slice and the trim

editcap copies a capture into a new
one, changed in one of a few honest
ways. The original is never touched.

**By packet range** — keep packets 1–5:

```bash
editcap -r lab.pcapng first-five.pcapng 1-5
```

(The `-r` means "keep the listed ones";
without it, editcap *removes* the
listed range.) `capinfos -c` confirms:
5 packets.

**By time** — keep everything at or
after a wall-clock moment:

```bash
editcap -A "2026-10-25 00:27:15" lab.pcapng late.pcapng
```

4 packets: the capture's final act (the
last two ping exchanges). `-B` sets the
end of the window. Time-slicing is how
a 4 GB overnight capture becomes the
ninety seconds around the fault that
someone can actually open — and how you
cut the parts of a file that contain
other people's business before sharing
it (chapter 14's rule: share the five
packets that prove the point).

editcap also strips duplicate packets
(`-D`), which rescues captures taken on
both ends of a link at once, and shifts
times (`-t`), which marries captures
from machines whose clocks disagreed —
use it knowing you are editing
evidence, and say so in the file's
comments.

## mergecap: several files, one timeline

```bash
mergecap -w merged.pcapng first-five.pcapng late.pcapng
```

Nine packets, in time order. mergecap
is the other half of the multi-machine
story: capture on the laptop *and* the
router, merge, and read one interleaved
conversation. Clock honesty decides
whether the result is evidence or
fiction — two machines disagreeing by a
second produce a merged file in which
answers precede questions, and editcap's
`-t` exists because of exactly that.

## text2pcap: resurrection for hex

Logs, old documentation, and some
network gear preserve packets as hex
dumps in text. text2pcap rebuilds a
capture from them:

```bash
text2pcap -o hex -e 0x800 hexdump.txt from-hex.pcapng
```

The lab fed it a single IPv4 header's
worth of hex (kept in
`wireshark-lab/hexdump.txt`) and got
back a one-packet file that tshark
dissects like any other. The `-e` tells
it which protocol the bytes begin with.
It is a niche tool that, the one time a
vendor's log holds your only copy of a
packet, is worth the whole chapter.

## House rules for a drawer of captures

- **Name files like evidence**:
  `2026-10-25-router-resets.pcapng`
  beats `cap3-final-FINAL.pcapng` in
  every month that follows.
- **Comment inside pcapng** (chapter
  11): what you were hunting, where the
  machine sat, what the capture filter
  was. The file will outlive your memory
  of the afternoon.
- **Convert deliberately**: most tools
  read pcapng now; when one cannot, save
  a `.pcap` copy rather than converting
  the original away.
