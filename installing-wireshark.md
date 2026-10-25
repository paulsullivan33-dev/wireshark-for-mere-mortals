# Installing Wireshark

Wireshark is free, open source, and
published by the Wireshark Foundation.
Get it from
[wireshark.org](https://www.wireshark.org/download.html)
or your system's package manager —
never from a download mirror you found
in a search ad, because a packet
analyser is exactly the sort of tool a
trojan would love to impersonate.

The install has two halves everywhere:
the program, and **permission to
capture**. Reading saved files needs no
special rights at all. Grabbing live
traffic does, because an interface in
capture mode hears everything addressed
to the machine — and the crew is
designed so that only tiny `dumpcap`
(chapter 2) holds that power.

## Windows

Download the 64-bit installer and run
it. Accept the offered **Npcap** install
when the wizard proposes it — that is
the capture library, and without it
Wireshark can open files but sees no
interfaces. Leave the USB capture option
off unless you know you need it. Any
ordinary user can then capture: Npcap's
service holds the rights, and your
account does not need to be an
administrator for everyday captures.
If the interface list is empty after
installing, the usual cause is a skipped
or failed Npcap — reinstall it from
[npcap.com](https://npcap.com) before
suspecting anything else.

## macOS

Install the disk image from Wireshark,
or `brew install --cask wireshark`.
Alongside the app comes **ChmodBPF**, a
small helper that, at each login, gives
the `access_bpf` group access to the
capture devices. Your account joins
that group during installation, which is
why a log-out-and-back-in may be needed
before the interface list fills. No
rights request should follow you after
that; if Wireshark asks to run as root,
something in the ChmodBPF setup has
failed, and re-running the installer is
the boring fix.

## Linux

Two packages cover most readers:

```bash
sudo apt install wireshark        # Debian, Ubuntu: window + tools
sudo apt install tshark           # command line only, for servers and small machines
```

(DNF and Pacman systems: `sudo dnf
install wireshark` / `sudo pacman -S
wireshark-qt`; the permission story
below is the same idea everywhere.)

Halfway through, Debian and Ubuntu ask
a pointed question: *"Should
non-superusers be able to capture
packets?"* Say **yes**. That answer
creates a `wireshark` group, makes
`dumpcap` executable by that group
only, and grants it exactly two Linux
capabilities — the right to open raw
sockets and to administer network
settings:

```bash
sudo usermod -a -G wireshark "$USER"   # then log out and back in
getcap "$(command -v dumpcap)"
```

Check: `getcap` answers with a line
ending `cap_net_admin,cap_net_raw=eip`
— the receipt that dumpcap, and only
dumpcap, holds capture rights. The whole
graphical program runs as you.

If you answered no at install time and
now meet "permission denied" at
capture (chapter 15), the re-answer is
`sudo dpkg-reconfigure wireshark-common`
on Debian/Ubuntu, followed by the
`usermod` line above.

## What this guide's lab did

The lab machine had no screen, so it
installed only the command-line half —
`tshark` plus its file tools — from
Ubuntu's repositories (the exact version
is recorded in `criteria.md`). Its
install ran unattended, which answered
the question above with *no*: no
`wireshark` group, no capabilities on
`dumpcap`. That accident turned out to
be a gift — the receipts are in
transcript t01 — because the lab then
applied this chapter's setup by hand
(group, ownership, mode, `setcap`) and
verified each receipt afterwards,
exactly as a reader who answered no
would. The lab itself ran as the root
user in a restricted sandbox, where
capture permission is never in question;
everything after installation was then
captured and read with tshark itself.

## A word before your first capture

A capture can contain passwords sent in
the clear, private messages, and other
people's traffic if you run it on a
shared segment. On your own machines
and your own network, that is your
business; on an employer's, a school's
or anyone else's network, get
permission first — in many places,
capturing traffic you are not party to
is against policy or law. Chapter 12's
recipes assume the home network you own.
