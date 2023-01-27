# Summary

This document specifies a private DHCP option for use at sites that make use
of the OpenDispense2 vending machine control software. The option allows
clients to learn the hostname of a local dispense server, and automatically
configure their local installation of OpenDispense2.

# Requirements

The "UCC Dispense Server Name" DHCP option is implemented identically to
the "Domain Search" option specified in
[RFC3397](https://www.rfc-editor.org/rfc/rfc3397.html), except for the
following:

- The option code MUST be selected in a site-specific manner from the
  private-use DHCP option codes (224 - 254). The option code SHOULD be
  selected so as not to conflict with any other private-use DHCP option codes
  in use within the same site.

- References to the "searchstring" are instead to be read as references to a
  "Hostname List".

- Currently, OpenDispense2 does not support the configuration of multiple
  servers. However, clients MUST be able to accept an option with multiple
  hostnames in the Hostname List. Unless additonal considerations\* apply,
  clients SHOULD select the first hostname in the list to configure
  OpenDispense2.

\* *For example, a future implementation may allow a user to prefer a
particular dispense server, or to select between multiple servers.*

- The "MUST" requirement in
[section 2 of RFC3397](https://www.rfc-editor.org/rfc/rfc3397.html#section-2)
to employ the path compression specified in
[section 4.1.4 of RFC1035](https://www.rfc-editor.org/rfc/rfc1035.html#section-4.1.4)
is relaxed to a "MAY".

*Rationale: the compression requirement in
[RFC3397](https://www.rfc-editor.org/rfc/rfc3397.html) was established because
it was anticipated that searchlists may contain many similar elements.
However, it is not anticipated that more than 1 or 2 dispense servers will
be configured on any particular network, thus reducing the potential benefit.
On the other hand, decoding the path compression adds complexity to the
implementation.*
