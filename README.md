# tinc

This is the README file for tinc version 1.1pre18. Installation
instructions may be found in the INSTALL file.

tinc is Copyright © 1998-2021 Ivo Timmermans, Guus Sliepen <guus@tinc-vpn.org>, and others.

For a complete list of authors see the AUTHORS file.

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or (at
your option) any later version. See the file COPYING for more details.

## Building

A detailed instruction on how to build tinc from source is available in [INSTALL.md](INSTALL.md).

## Nightly builds

You can download pre-built binary packages for multiple Linux distributions and Windows here:

- [development version](https://github.com/gsliepen/tinc/releases/tag/latest)
- [latest release](https://github.com/gsliepen/tinc/releases/latest)

Note that these packages have not been heavily tested and are not officialy supported by the project. Use them at your own risk. You are advised to use tinc shipped by your distribution, or build from source.

## This is a pre-release

Please note that this is NOT a stable release. Until version 1.1.0 is released,
please use one of the 1.0.x versions if you need a stable version of tinc.

Although tinc 1.1 will be protocol compatible with tinc 1.0.x, the
functionality of the tinc program may still change, and the control socket
protocol is not fixed yet.

## Security statement

This version uses an experimental and unfinished cryptographic protocol. Use it
at your own risk.

When connecting to nodes that use the legacy protocol used in tinc 1.0, be
aware that any security issues in tinc 1.0 will apply to tinc 1.1 as well. On
September 6th, 2018, Michael Yonly contacted us and provided proof-of-concept
code that allowed a remote attacker to create an authenticated, one-way
connection with a node using the legacy protocol, and also that there was a
possibility for a man-in-the-middle to force UDP packets from a node to be sent
in plaintext. The first issue was trivial to exploit on tinc versions prior to
1.0.30, but the changes in 1.0.30 to mitigate the Sweet32 attack made this
weakness much harder to exploit. These issues have been fixed in tinc 1.0.35
and tinc 1.1pre17. The new protocol in the tinc 1.1 branch is not susceptible
to these issues. However, be aware that SPTPS is only used between nodes
running tinc 1.1pre\* or later, and in a VPN with nodes running different
versions, the security might only be as good as that of the oldest version.

## Compatibility

Version 1.1pre18 is compatible with 1.0pre8, 1.0 and later, but not with older
versions of tinc.

When the ExperimentalProtocol option is used, tinc is still compatible with
1.0.X, 1.1pre11 and later, but not with any version between 1.1pre1 and
1.1pre10.

## Requirements

In order to compile tinc, you will need a GNU C compiler environment. Please
ensure you have the latest stable versions of all the required libraries:

- LibreSSL (http://www.libressl.org/) or OpenSSL (https://openssl.org/) version 1.1.0 or later.

The following libraries are used by default, but can be disabled if necessary:

- zlib (https://zlib.net/)
- LZO (https://www.oberhumer.com/opensource/lzo/)
- ncurses (https://invisible-island.net/ncurses/)
- readline (https://cnswww.cns.cwru.edu/php/chet/readline/rltop.html)

## Features

Tinc is a peer-to-peer VPN daemon that supports VPNs with an arbitrary number
of nodes. Instead of configuring tunnels, you give tinc the location and
public key of a few nodes in the VPN. After making the initial connections to
those nodes, tinc will learn about all other nodes on the VPN, and will make
connections automatically. When direct connections are not possible, data will
be forwarded by intermediate nodes.

Tinc 1.1 support two protocols. The first is a legacy protocol that provides
backwards compatibility with tinc 1.0 nodes, and which by default uses 2048 bit
RSA keys for authentication, and encrypts traffic using AES256 in CBC mode
and HMAC-SHA256. The second is a new protocol which uses Curve25519 keys for
authentication, and encrypts traffic using Chacha20-Poly1305, and provides
forward secrecy.

Tinc fully supports IPv6.

Tinc can operate in several routing modes. In the default mode, "router", every
node is associated with one or more IPv4 and/or IPv6 Subnets. The other two
modes, "switch" and "hub", let the tinc daemons work together to form a virtual
Ethernet network switch or hub.

Normally, when started tinc will detach and run in the background. In a native
Windows environment this means tinc will install itself as a service, which will
restart after reboots. To prevent tinc from detaching or running as a service,
use the -D option.

The status of the VPN can be queried using the "tinc" command, which connects
to a running tinc daemon via a control connection. The same tool also makes it
easy to start and stop tinc, and to change its configuration.
