# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.


## Verification notes for this guide

Built and verified 2026-10-24/25 against
**Wireshark 4.2.2** (Ubuntu package
4.2.2-1.1build3), tshark, dumpcap and
the file tools as shipped in Ubuntu
24.04, on a sandbox machine running
Ubuntu 24.04 (the capture file itself
records the lab machine's CPU and
kernel, shown by capinfos in transcript
t08).

**Primary sources read, not remembered:**
Wireshark's official user guide and man
pages for installation and capture
permissions (the wireshark group,
dumpcap capabilities and the Debian
install question in chapter 3), display
and capture filter syntax (chapters 6
and 7), tshark's `-z` statistics and
field output (chapter 9), the file
tools' options (chapter 10), and TLS
key-log decryption and monitor mode
(chapter 14).

**The sandbox, declared.** The lab
machine was a restricted sandbox with
**no display**, and the guide's lab ran
as its root user. Three consequences,
named wherever they matter: the
**graphical window was never run**
(installing it would have proved
nothing without a screen to show it);
captures were taken on the **loopback
interface**, the only seat whose
traffic the lab could generate and
explain end to end; and capture
**permission was never in question**
for the lab's own runs — the setup in
chapter 3 is verified as configuration
(see below), not exercised by an
unprivileged capture. Those are the
sandbox's properties, not Wireshark's.

**Ran for real** (transcripts in
`wireshark-lab/transcripts/`):

- t00 — `tshark --version` → 4.2.2;
  `tshark -D` lists the interfaces,
  loopback among them; `dumpcap
  --version` agrees.
- t01 — the unattended install's
  receipts: `install-setuid: false`, no
  wireshark group, no capabilities on
  dumpcap; then the documented setup
  applied by hand (system group
  created, dumpcap `root:wireshark`
  mode 0750, `setcap
  cap_net_admin,cap_net_raw+eip`) and
  each receipt verified with `getent`,
  `ls -l` and `getcap`.
- t02/t03 — a 12-second loopback
  capture (`lab.pcapng`, kept in the
  repo): 46 packets — 8 ICMP (four
  ping exchanges; the first run's
  first exchange predates the file's
  first frame, visible in the sequence
  numbers), three complete HTTP
  fetches against a throwaway local
  web server (200, 200, 404), and a
  SYN to port 9 answered by RST.
- t04 — frame 8 (the first GET)
  dissected in full with `-V`; the
  chapter 5 text quotes its real
  fields, including the raw window
  (64) beside its calculated size
  (65536) and the mid-conversation
  completeness verdict.
- t05 — display filters with counts:
  `icmp` 8, `http` 6, `tcp.port ==
  8080` 36, `frame.len > 200` 6, the
  reset filter returning exactly frame
  42, the 404 filter exactly frame 36,
  and `not tcp and not icmp` empty.
- t06 — `follow,tcp,ascii,0`
  reassembles the first fetch into
  readable request and response;
  `conv,tcp` (four conversations),
  `endpoints,ipv4` (the loopback
  double-count, noted in chapter 8),
  and `io,phs` (46 frames: 8 ICMP, 38
  TCP, 6 HTTP).
- t07 — `-T fields` extraction of the
  three requests and of ICMP types and
  sequences; `-T json` of frame 42.
- t08 — `capinfos` on the lab file
  (46 packets, 2.37 s, pcapng, hashes);
  `editcap -r` keeping packets 1–5
  (5 in the result); `editcap -A` by
  wall-clock time (4 kept); `mergecap`
  joining the slices (9); `text2pcap`
  rebuilding a one-packet file from
  the hex header kept in
  `wireshark-lab/hexdump.txt`.
- t09 — a second capture under the BPF
  capture filter `icmp`, run while a
  web fetch happened alongside: the
  file holds exactly 4 packets, all
  ICMP — the filter acted at capture,
  as chapter 7 claims.

**Errors hit and diagnosed for real,**
written up where they belong: the
unattended install's skipped permission
setup (t01, chapter 3); a first editcap
attempt with a packet-range argument
where a time window was meant, which
silently produced a file the next step
could not use — the corrected `-A`
form is what chapter 10 teaches and
t08 keeps.

**Documented only, and labelled as
such in the chapters:** the graphical
window — its panes, menus, right-click
filter building, colouring-rule editor
and statistics windows (the same engine
and statistics were all exercised
through tshark, but no pixel of the
window was rendered by the lab);
installing on Windows and macOS,
including Npcap and ChmodBPF behaviour;
an unprivileged user's first capture
under the group setup (verified as
configuration only, per above);
capturing on a real Ethernet or Wi-Fi
interface, mirror ports, taps and
monitor mode; TLS decryption with a
key log (no TLS traffic was generated;
chapter 14 teaches the mechanism from
the documentation); DNS, VoIP and other
protocols absent from the loopback
capture (recipes in chapter 12 that
reach beyond the lab file are written
from the documentation and the filters
they use were verified syntactically
against the lab's tshark); profiles
and colouring rules in use (chapter 11
is documentation-based — they are
window furniture the headless lab could
not exercise); and long ring-buffer
captures (the dumpcap command is given,
not run for hours).

**Fixtures:** everything the lab made
up is small, generic, and in the repo:
two static web pages for a fictional
loopback-only tea room, a one-line hex
header, and traffic between a machine
and itself. Every address in every
capture is 127.0.0.1. Nothing in the
lab is modelled on any real person,
organisation or event.
