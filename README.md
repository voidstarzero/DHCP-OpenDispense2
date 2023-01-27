# DHCP-OpenDispense2
Custom DHCP option to configure OpenDispense2 client information 

# What?

[OpenDispense2](https://gitlab.ucc.asn.au/UCC/OpenDispense2) is a software
package written to allow members of the [University Computer Club]
(https://www.ucc.asn.au/) to order drinks from the club's vending machine
from the comfort of their own terminal.

This repo contains a set of configuration customizations and helper scripts to
allow clients to automatically learn the address of the `dispsrv` server over
DHCP.

# How?

This project specifies a private DHCP option `ucc-dispense-server-name`, which
encodes the hostname of a dispense server that exists on the network. It also
provides instructions and snippets to help deploy the option on a Linux-based
network.

Read more in the [SPECIFICATION.md](SPECIFICATION.md).

# Do I want this?

Unless you're a UCC member, probably not. That said, the full OpenDispense2
suite is open source. So, if you *do* happen to have a spare serial or
modbus-based vending machine lying around, go right ahead. You'll probably
want to customize quite a bit of the configuration, including the private
DHCP option number, to suit your environment.

# Requirements

## Server

This guide assumes the use of ISC's DHCP sever `dhcpd`. However, any DHCP
server that can be configured to serve custom options should be suitable.

If you're just here to set up a client to work with an existing DHCP server
with the option already configured, the server version in use shouldn't
matter, anyway.

## Clients

- `dhclient`
- `bash`

The scripts provided only work with ISC' DHCP client `dhclient`. However, a
similarly-configurable DHCP client with both:

- The ability to request and receive private options; and
- The ability to add hooks that can read received DHCP options.

I believe Roy Marples' fantastic `dhcpcd` would be suitable, though I haven't
tried to write the necessary hooks.

### NetworkManager

NetworkManager's internal DHCP server lacks the configurability required to
use private options. Thankfully, NetworkManager allows delegation of
DHCP duties to either `dhclient` or `dhcpcd`. Add the following to your
`/etc/NetworkManager/NetworkManager.conf`:

```
[main]
dhcp=dhclient
```

and then follow the steps provided for `dhclient`.

# Configuring the DHCP server

**1)** Add the following line near the top of `/etc/dhcp/dhcpd.conf`, outside
  any sections. Replace `<code>` with a site-specific private DHCP option
  number.

```
option ucc-dispense-server-name code <code> = domain-list;
```

**2)** In the `subnet` blocks corresponding to networks that are meant to
  advertise the dispense server, add the following option line (with the
  dispense server's hostname inserted):

```
subnet X.X.X.X netmask Y.Y.Y.Y {
    ...
    option ucc-dispense-server-name "hostname.of.my.dispense.server";
    ...
}
```

Ensure that the given hostname will be resolvable by clients that are on the
target networks.

# Configuring a DHCP client

**1)** Add the following line near the top of `/etc/dhcp/dhclient.conf`,
  outside any sections. Replace `<code>` with the code selected by the DHCP
  server administrator in the previous step.

```
option ucc-dispense-server-name code <code> = domain-list;
```

**2)** Locate the line that starts with `request ...` and add the
  `ucc-dispense-server-name` option, like so:

```
request subnet-mask, broadcast-address, ...
    ..., ntp-servers, ucc-dispense-server-name;
```

**3)** Copy the `dhclient` hook script to `/etc/dhcp/dhclient-exit-hooks.d`:

```
$ cp hooks/dhclient /etc/dhcp/dhclient-exit-hooks.d/99-ucc_dispense_server_name
```

# Future steps

- Write hook scripts for `dhcpcd`.
- Customize client installation script based on the DHCP client in use.
- Automatically configure NetworkManager as required above.