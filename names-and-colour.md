# Names and Colour

A raw capture is numbers: addresses,
ports, hex. Two kinds of decoration turn
it into something a human can patrol —
**names**, which translate numbers into
words, and **colouring rules**, which
paint packets by kind so the odd ones
stand out before you read a word. Both
are display furniture (chapter 2): they
change nothing in the file, and both can
mislead if you forget they are switched
on.

## Name resolution

Wireshark can resolve three things, each
with its own switch in View → Name
Resolution:

- **MAC addresses** to makers and, where
  your hosts file or local tables know
  them, to names (`08:00:27:…` becomes
  `printer`). Harmless and usually worth
  having on a home network.
- **IP addresses** to hostnames, through
  DNS. Convenient — and it *generates
  DNS traffic of its own*, and can record
  names that were true when you opened
  the file rather than when the packets
  flew. On a capture meant as evidence,
  leave it off and note addresses.
- **Ports** to service names: 8080
  becomes `http-alt`. A name here is a
  guess from a table, not a fact about
  the packet — port 8080 carrying
  something that is not HTTP will still
  wear the label until the dissector
  speaks.

The transport-name trick worth knowing:
the *conversations* window (chapter 8)
lets you assign a name to an address for
this file, and Wireshark can store
per-capture name notes inside a pcapng.
For a home network, the sanest habit is
a small personal table — a hosts file of
your own devices (`192.168.1.20` is the
laptop, `.1` the router) — so names in
your captures are *yours*, stable and
true, rather than whatever DNS felt like
that afternoon.

## Colouring rules

Wireshark ships with a sensible palette:
errors and TCP trouble in angry colours,
checksums complaining in black on red,
ordinary chatter in pastels. The rules
are display filters (chapter 6) with a
colour attached, tested top to bottom;
the first match paints the packet.

Edit them under View → Coloring Rules.
Three additions earn their place on most
home desks:

| Rule (filter) | Colour idea | Why |
|---|---|---|
| `tcp.analysis.flags` | strong highlight | All TCP trouble at a glance |
| `dns.flags.response == 0` | mild highlight | Queries stand out from replies |
| `ip.addr == 192.168.1.20` | bright background | The machine under investigation, unmissable |

Two rules about the rules. First, colour
is an *opinion*: a red packet is a packet
a rule disliked, and chapter 15 lists the
classic heart-skip — a capture opened
with someone else's profile, wearing
someone else's alarms. Second, checksum
colours cry wolf on any machine with
**checksum offloading** (most of them):
the capture is taken before the network
card fills the checksum in, so perfectly
healthy packets show as broken. The
giveaway is that only *outgoing* packets
from the capturing machine are "wrong".

## Marking, ignoring, and comments

Three small tools finish the furniture.
**Marking** (Ctrl+M) pins a packet with
a dark background so you can find your
way back to it; marks live in the file
session, and the *Go to marked packet*
commands hop between them. **Ignoring**
a packet (Ctrl+X) removes it from counts
and statistics without deleting it —
useful when three packets of line noise
are wrecking a graph. A pcapng can also
carry **comments** on individual packets
(Edit → Packet Comment): "this is where
the user pressed the button" is worth
more, three weeks later, than any other
sentence in the file.

## Profiles tie it together

Everything in this chapter — columns,
filters, colours, resolution habits —
saves into the current **profile**
(chapter 2). Working on the home LAN?
Keep a `Home` profile with your device
names coloured and your noise hush-list
as the default filter. The profile name
shows in the window's status bar and in
tshark's `-C` option, and chapter 15's
first question — "am I looking at this
file through someone else's furniture?"
— is answered there.
