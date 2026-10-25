# TShark at the Command Line

tshark is the whole engine without the
window: the tool for headless machines,
for scripts, and for questions shaped
like "how many, which ones, give me a
list". Everything in this chapter was
run on the lab file; the transcripts
(t05–t07) hold the unedited output.

## The three shapes of a tshark call

```bash
tshark -i lo -a duration:10 -w out.pcapng   # capture (chapter 4)
tshark -r out.pcapng                        # read: the packet list
tshark -r out.pcapng -Y "http" -T fields …  # interrogate
```

Reading defaults to the summary list
from chapter 5. `-V` grows each packet
into its full tree (chapter 5's view);
`-x` adds the hex; `-2` reads the file
in two passes, which the analysis
annotations prefer when accuracy matters
more than speed. The `-q` flag quiets
the list when only statistics are
wanted, and nearly every serious tshark
call wears it.

## Filters, wearing both hats

`-Y` is the display filter (chapter 6);
`-f` is the capture filter (chapter 7)
and only makes sense while capturing.
The pair in one command is legal and
common: record everything on the
interface, *show* only the interesting
part as it flies past — though when a
file is being written, the file always
gets what `-f` allowed, never what `-Y`
showed.

## Fields: the spreadsheet maker

`-T fields` with one `-e` per field
prints a table of raw values, one line
per matching packet:

```bash
$ tshark -r lab.pcapng -Y "http.request" -T fields \
    -e frame.number -e http.request.method -e http.request.uri
8   GET  /
20  GET  /menu.html
32  GET  /gone.html
```

This is where tshark stops being a
viewer and becomes plumbing: the output
pipes into `sort`, `uniq -c`, `awk`, a
spreadsheet, or a script. Questions like
"every DNS name this machine asked
about, deduplicated" (chapter 12) are
one pipeline. `-E separator='|'` and
friends civilise the output for tools
that dislike tabs; `-T json` (or `ek`
for Elasticsearch-flavoured JSON)
emits the whole dissection as structured
data instead (transcript t07 shows
frame 42 as JSON) for programs, not
people.

Useful field names are discovered the
same way as filter names (chapter 6):
they are the tree's field names, and
`tshark -G fields` dumps the entire
dictionary — tens of thousands of
entries — for grepping on a rainy day.

## Statistics with -z

The `-z` family prints the window's
Statistics menu as text. The four this
guide leans on:

| Option | Prints |
|---|---|
| `-z io,phs` | Protocol hierarchy (chapter 8) |
| `-z conv,tcp` / `conv,udp` | Conversation table |
| `-z endpoints,ipv4` | Per-address totals |
| `-z follow,tcp,ascii,N` | Stream N, reassembled |

There are dozens more — per-protocol
service response times, HTTP request
counts by host, DNS by type — listed in
the tshark manual under `-z`. Each
accepts a display filter after a comma
(`-z conv,tcp,ip.addr==…` in spirit), so
a statistic can be about a *subset*:
the conversations of one sick machine,
not the world.

## Long captures without a human

dumpcap, tshark's capturing half, is
the right tool for unattended work
because it skips dissection entirely.
Its ring buffer is the pattern:

```bash
dumpcap -i eth0 -b filesize:100000 -b files:10 -w watch.pcapng
```

Ten files of about 100 MB each, rolling:
when the eleventh is needed, the oldest
is deleted. Something breaks at 3 a.m.;
you stop the capture at 9 and the
night's last gigabyte is waiting.
Chapter 7 explained why the capture
filter beside it should stay modest;
chapter 14 explains the disk arithmetic
that chooses the numbers.

Check: `tshark -r lab.pcapng -q -z
io,phs` and the window's Protocol
Hierarchy on the same file agree to the
packet — the same dissectors keep both
sets of books, which is the point of the
crew being one crew (chapter 2).
