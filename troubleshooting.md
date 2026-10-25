# Troubleshooting

Symptom first, then the cause the lab
or the documentation supports, then the
fix. Wireshark's troubles divide
honestly into four families: it cannot
start, it captures nothing, it captures
the *wrong* thing, and it shows you
something alarming that is not real.

## It cannot start capturing

**"You don't have permission to capture
on that device."**
dumpcap lacks its rights. On Linux this
is chapter 3's unfinished business: the
install-time question answered *no*, or
your account missing from the `wireshark`
group, or the capabilities lost.
`getcap "$(command -v dumpcap)"` should
answer `cap_net_admin,cap_net_raw=eip`;
`dpkg-reconfigure wireshark-common`
(Debian/Ubuntu) plus a log-out-and-in
is the standard repair. On Windows, the
same symptom means Npcap is missing or
its service stopped; reinstall Npcap.

**The interface list is empty (or the
one you want is missing).**
The program runs but no capture library
found anything to offer. Windows:
Npcap again — it *is* the interface
list. macOS: the ChmodBPF helper is not
running or your account is not in
`access_bpf`; re-running the installer
fixes both. Linux virtual interfaces
(on a VM, a container) show only what
the host delegates — the interface may
genuinely not be yours to capture.

**Wireshark itself demands root.**
It shouldn't, ever, on a healthy
install: the design keeps rights inside
dumpcap alone. Treat a Wireshark that
"only works as root" as a broken
install to repair (above), not a way of
life — a dissector parsing hostile
traffic with root rights is a risk the
crew was split precisely to avoid.

## It captures nothing — or the wrong things

**The capture runs and stays empty.**
In order: the capture filter rejects
everything (clear it and retry —
chapter 7's bet, lost); the interface
is the wrong one (a laptop can list a
dozen; the traffic is on another); the
traffic genuinely never reaches this
seat — the switched-network rule of
chapter 14, the most common cause of
all and nobody's fault; or, on Wi-Fi,
the adapter hears only its own traffic
because monitor mode is off or
unsupported on this system.

**It shows only broadcasts and my own
traffic.**
Not a fault: that *is* a switched
network from one seat (chapter 14).
Mirror port, tap, or move the capture
onto a machine in the conversation.

**Everything appears twice.**
Two honest causes. You are capturing on
**both ends** of a virtual path — the
`any` interface on Linux counts an
in-machine conversation once per
interface it crosses. Or a bridge/tap
arrangement forwards copies. Compare
frame timestamps: true duplicates agree
to the nanosecond. (The lab's loopback
capture did *not* double its packets —
some systems do; tshark's file tools
editcap `-D` remove true duplicates
when a capture comes back doubled.)

**The other protocol is missing.**
You expected DNS and the file holds
none: the client cached the answer
before you started (restart the app,
flush the cache), or it uses encrypted
DNS (DoH/DoT) and the lookups are inside
TLS to a resolver — the hierarchy table
(chapter 8) showing *only* TLS where
DNS should be is the tell.

## It shows alarming things that are not real

**"Bad checksum" on half the file.**
Checksum **offloading**: the capturing
machine's own outgoing packets were
copied before its network card computed
the checksums. Only local, only
outgoing, everything else fine. Confirm
by the pattern, then ignore — or tick
the dissector's "don't validate" option
in your profile (chapter 11).
Reassuringly, the lab's loopback capture
labels its checksums `[unverified]`
rather than wrong.

**"Malformed packet" everywhere.**
Often the dissector, not the packet: a
new protocol version read by an older
Wireshark. Check the packet's bytes
agree with *some* sensible reading,
update Wireshark, and be slow to accuse
the sender. The opposite pattern — one
genuinely truncated packet — usually
means a snap-length limit at capture
(chapter 5's mismatched lengths).

**A filter that "can't be wrong"
matches nothing.**
Chapter 7's VLAN trap on the capture
side; on the display side, the usual
suspects are a field that exists only
in a layer you are not looking at
(`http` on a TLS conversation), or the
wrong file's profile furniture
(chapter 11). The dissection tree is
the dictionary: right-click the field
you can *see* and let it write the
filter.

**Packets are missing that you watched
happen.**
A busy capture drops some — dumpcap
reports the count when it stops
("packets dropped" in the capture
summary). Absence in the file is not
absence on the wire (chapter 1); on a
loaded link, tighten the capture filter
or accept the sampling.

Check: most mysteries in this chapter
yield to the same three questions —
*which seat, which filter, whose
furniture* — asked in that order.
