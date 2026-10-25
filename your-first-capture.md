# Your First Capture

Time to take a real one. This chapter
captures the smallest honest network
there is — a machine talking to itself —
because every packet in it can be
explained, and because the loopback
interface exists on every machine you
will ever sit at. Nothing here needs
permission beyond your own computer.

## In the window

Open Wireshark. The welcome screen lists
interfaces with tiny sparkline graphs of
their traffic. Double-click the one
named **Loopback** (on Windows it may be
called *Adapter for loopback traffic
capture*). Packets begin to arrive —
on an idle machine, slowly: your
computer chatting to itself about
updates, time and discovery. The red
square stops the capture; the green fin
restarts it. File → Save writes it as a
`.pcapng`. That is the whole of a first
capture; the rest of this chapter is the
same act with the training wheels named,
and then without the window at all.

## On the command line, and the lab's capture

```bash
tshark -D                              # find the interface's number or name
tshark -i lo -a duration:10 -w myfirst.pcapng
```

`-i` chooses the interface; `-a
duration:10` stops on its own after ten
seconds — a capture with a planned end
beats one you forget about; `-w` writes
the file. While it runs, *make something
happen*: the guide's lab started a tiny
web server on the machine itself and
then, inside a 12-second loopback
capture, did four deliberate things:

```bash
ping -c 3 127.0.0.1
curl http://127.0.0.1:8080/            # a page that exists
curl http://127.0.0.1:8080/gone.html   # a page that doesn't: a real 404
curl http://127.0.0.1:9/               # a port with nobody home: refused
```

The harvest (transcript t02) was **46
packets**: the pings, three complete web
conversations, and one two-packet
refusal. Small enough to read whole —
which the next two chapters do.

Why generate traffic on purpose? A first
capture of ambient noise teaches nothing
but scrolling. A capture whose contents
you *caused* — you know the ping was
yours, the 404 was yours — lets you
check Wireshark's story against your
own memory, which is the fastest way to
learn what the panes are telling you.
From here on, "the lab capture" means
this file: `wireshark-lab/lab.pcapng`,
and its transcripts sit in
`wireshark-lab/transcripts/` if you
want to see the commands unedited.

## The details that matter already

**Stop conditions are a kindness.**
Besides `duration:`, tshark and dumpcap
accept `filesize:` and `packets:` —
learn to always give a long capture one
(chapter 10's ring buffer is the grown-up
form). An unbounded capture on a busy
machine is a disk-filling accident with
a fuse already lit.

**The file is the unit of work.** Notice
that capturing and reading were separate
commands: nothing about the 46 packets
needed the window, the machine that
captured them, or even the same day.
That is the workflow the rest of the
guide assumes — capture where the
traffic is, read where the human is.

**Loopback is a special seat.** Both
ends of every conversation are
127.0.0.1, the "Ethernet" header is a
formality filled with zeros, and a
conversation's two directions can only
be told apart by ports. Real interfaces
fix all of that — real addresses, real
MACs — and chapter 5 notes each place
where loopback simplified the picture.

Check: `capinfos lab.pcapng` reports 46
packets spanning about 2.4 seconds —
the four caused events, and nothing
else. If your own first capture shows
thousands of packets, you captured a
real interface (also fine, and chapter
12 is written for exactly that file).
