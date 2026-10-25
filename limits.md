# Limits

Stated plainly, as this series insists:
what Wireshark cannot do, cannot see, or
should not be asked to do.

## It sees one seat on the network

A capture is the view from where you
sat. On a switched network — every
modern home network — the switch sends
each frame only to its destination, so
a capture on your laptop holds your
laptop's conversations and the
broadcast chatter, not the television's
or the NAS's. No setting changes this;
the frames never arrive. The ways
around it are physical:

- Run the capture **on** one of the
  machines involved (a server, a router).
- Use a **mirror (SPAN) port** on a
  managed switch, which copies other
  ports' traffic to yours.
- Insert a **tap**, a small hardware
  listener, on the cable in question.

Each has costs — mirror ports can drop
copies under load and cheap switches
lack them; taps cost money and a cable
break to install. On Wi-Fi the picture
inverts: the radio hears everything in
reach, but almost all of it is
encrypted per-client, and **monitor
mode** (capturing other stations'
frames raw) depends on your adapter and
driver, works well on some Linux
setups, poorly or not at all on most
Windows machines, and still yields
ciphertext without the network key and
a captured handshake.

## Encryption is the big wall

Most traffic that matters is TLS now.
Wireshark shows the ceremony — the
handshake, certificates, the server's
name, sizes and timings — and then the
conversation becomes numbered boxes of
ciphertext, correctly and by design.
Options, in order of realism:

- **Read around it.** Metadata answers
  more questions than people expect:
  who talked, when, how much, how fast,
  whether the handshake failed (chapter
  12's slow-page recipe lives here).
- **Use the endpoint's keys.** Browsers
  and some programs can log TLS session
  keys (the `SSLKEYLOGFILE` mechanism);
  a capture taken with that log can be
  decrypted in Wireshark. This works
  only for traffic *you* control the
  endpoint of, which is exactly the
  legal and sensible boundary anyway.
- **Move the seat.** Capture inside the
  endpoint — on the server after TLS
  terminates, or with a proxy
  (mitmproxy, chapter 13) the client
  trusts on purpose.

There is no fourth option, and anyone
selling one is selling something else.

## Speed and scale

A capture is a firehose with a notebook.
On a gigabit link kept busy, frames
arrive faster than disks — and far
faster than dissectors — can work;
dumpcap will report **dropped packets**,
and a dropped packet is indistinguishable
in the file from one that never
happened. Fast-link work therefore
captures with tight capture filters
(chapter 7), to files, with dissection
left for later — or moves to summary
tools (chapter 13's Zeek) entirely.
File sizes follow the same arithmetic:
an unfiltered hour of a busy household
can be gigabytes, and Wireshark the
window grows slow and memory-hungry
with multi-gigabyte files. Chapter 10's
trim-and-slice tools are the answer this
guide can offer; beyond a few gigabytes,
divide the capture rather than opening
it whole.

## One protocol stack deep, no opinions included

Wireshark decodes protocols it knows —
several thousand — and displays the
rest as data. A proprietary protocol, a
brand-new one, or one running inside an
unexpected tunnel may stop at the layer
where recognition stopped. The
dissectors are also software: a
"malformed packet" line sometimes means
the *dissector* misread a legal packet,
especially on new protocol versions
(chapter 15). And Wireshark has no
opinion about intent — a port scan and
a broken monitoring script can look
remarkably alike until a human who
knows the network reads them.

## The legal and human limit

Recording traffic records people: their
devices, habits, and anything sent
unencrypted. On your own network, about
your own machines, that is diagnosis.
Anywhere else — an employer's network,
a customer's, a family's you do not
administer, anything you capture on a
shared medium — get permission in
advance and expect rules (laws differ;
policies are usually stricter). Keep
captures like the sensitive documents
they are: share the five packets that
prove the point, not the whole file,
and delete captures when the question
they answered stays answered.
