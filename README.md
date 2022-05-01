# ansible-tor

Installs Tor relays on Debian-based systems.

This role uses the default Tor instance (configuration in _/etc/tor/torrc_)
for hidden services and optionally a SOCKS proxy. Bridge, middle and exit
relays are created as additional instances using the _tor-instance-create_
script (configuration under _/etc/tor/instances_).

## Requirements

- Debian-based system with systemd
- Offline keys as described below

## Role Variables

Please see [defaults/main.yml](defaults/main.yml) for default values.

### Main Variables

<table>
<tr>
  <th>Variable</th>
  <th>Description</th>
</tr>
<tr>
  <td>tor_contact_info</td>
  <td><tt>ContactInfo</tt> value.</td>
</tr>
<tr>
  <td>tor_instances_bridge</td>
  <td>Bridge instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_instances_middle</td>
  <td>Middle instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_instances_exit</td>
  <td>Exit instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_offline_keys</td>
  <td>Local base directory for offline keys (see below).</td>
</tr>
</table>

### Other Variables

<table>
<tr>
  <th>Variable</th>
  <th>Description</th>
</tr>
<tr>
  <td>tor_apparmor</td>
  <td>
    Use <tt>false</tt> to disable AppArmor, in case Tor is running in an LXC
    container. (<a href="https://gitlab.torproject.org/tpo/core/tor/-/issues/17754">Bug 17754</a>)
  </td>
</tr>
<tr>
  <td>tor_avoid_disk</td>
  <td>Set <tt>AvoidDiskWrites</tt>, for flash-based systems.</td>
</tr>
<tr>
  <td>tor_default_client_onion_auth</td>
  <td>Client authorizations for the default instance (see below).</td>
</tr>
<tr>
  <td>tor_default_hidden_services</td>
  <td>Hidden service definitions for the default instance (see below).</td>
</tr>
<tr>
  <td>tor_default_socks_port</td>
  <td>SOCKS port of the default instance.</td>
</tr>
<tr>
  <td>tor_dist</td>
  <td>Distribution name on <a href="https://deb.torproject.org/torproject.org/dists/">deb.torproject.org</a>.</td>
</tr>
<tr>
  <td>tor_exit_policy</td>
  <td>
    List of <tt>ExitPolicy</tt> values (exits only).
    Overrides <tt>tor_reduced_exit_policy</tt>.
  </td>
</tr>
<tr>
  <td>tor_instance_settings</td>
  <td>List of additional settings for bridge, middle and exit instances.</td>
</tr>
<tr>
  <td>tor_my_family</td>
  <td>List of fingerprints for <tt>MyFamily</tt>.</td>
</tr>
<tr>
  <td>tor_nameservers</td>
  <td>
    List of nameserver addresses to add to <i>/etc/tor/resolv.conf</i>, which
    will be used by bridge, middle and exit instances as
    <tt>ServerDNSResolvConfFile</tt>.
  </td>
</tr>
<tr>
  <td>tor_onion_auth</td>
  <td>Local directory for client authorization files (see below).</td>
</tr>
<tr>
  <td>tor_packages_extra</td>
  <td>Extra system packages to install.</td>
</tr>
<tr>
  <td>tor_reduced_exit_policy</td>
  <td><tt>ReducedExitPolicy</tt> value (exits only).</td>
</tr>
<tr>
  <td>tor_repository</td>
  <td>Use the Tor Project package repository.</td>
</tr>
</table>

## Instance Configuration

Bridge, middle and exit relays are created using the _tor-instance-create_
script and configured via three separate variables in the inventory (see above).

The following properties are common to all instance types (required properties
in **bold**):

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td><b>name</b></td>
  <td>
    Instance name. Example: the instance name <tt>foo</tt> will create the
    systemd unit <tt>tor@foo</tt> owned by user/group <tt>_tor-foo</tt>.
    The configuration will be in <i>/etc/tor/instances/foo/torrc</i>, the data
    in <i>/var/lib/tor-instances/foo</i>.
  </td>
</tr>
<tr>
  <td><b>or_ports</b></td>
  <td>List of <tt>ORPort</tt> values.</td>
</tr>
<tr>
  <td>nickname</td>
  <td>Relay nickname.</td>
</tr>
<tr>
  <td>relay_bandwidth_rate</td>
  <td><tt>RelayBandwidthRate</tt> value.</td>
</tr>
<tr>
  <td>relay_bandwidth_burst</td>
  <td><tt>RelayBandwidthBurst</tt> value.</td>
</tr>
</table>

### Bridge Configuration

The following properties are used by bridge instances (required properties in
**bold**):

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td>ext_or_port</td>
  <td><tt>ExtORPort</tt> value.</td>
</tr>
<tr>
  <td><b>pluggable_transports</b></td>
  <td>List of pluggable transport definitions (see below).</td>
</tr>
</table>

Example:

```yaml
tor_instances_bridge:
  - name: bridge
    nickname: MyCoolBridge
    or_ports: [9000]
    pluggable_transports:
      - name: obfs4
        exec: /usr/bin/obfs4proxy
        address: 0.0.0.0:8443
```

### Middle/Exit Configuration

The following properties are used by middle/exit instances:

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td>address</td>
  <td>
      The IPv4 address of this server.
  </td>
</tr>
<tr>
  <td>ipv6_exit</td>
  <td>Allow IPv6 exit traffic (exits only).</td>
</tr>
<tr>
  <td>outbound_addresses</td>
  <td>
      List of <tt>OutboundBindAddress</tt> values.
  </td>
</tr>
</table>

Example:

```yaml
tor_instances_exit:
  - name: exit
    nickname: MyCoolExit
    or_ports:
      - 443
      - "[abcd::1:2:3:4]:443"
    ipv6_exit: 1

tor_exit_policy:
  - "accept *:80"
  - "accept *:443"
  [...]
  - "reject *:*"
```

### Exit Policy Blocks

On exit instances, the file */etc/tor/exit-policy-blocks* will be included in
instance torrc files before any `tor_exit_policy` values. You can use this to
place `ExitPolicy reject` statements in front of your exit policy.

## Offline Keys

A given host's and instance's offline keys are copied from the local directory
`{{ tor_offline_keys }}/{{ inventory_hostname }}/INSTANCE_NAME/keys`.

You can use the included script [tor-keygen](scripts/tor-keygen) to create and
update offline keys.

**Please note:** Tor creates the RSA key `secret_id_key` on new relays. This
key is part of the relay identity, so you should create a backup. If the key is
found in the local directory mentioned above, it is also copied to the host.

## Hidden Services

Hidden services are only configured on the default instance via the
`tor_default_hidden_services` variable.

The following properties are used by hidden services (required properties in
**bold**):

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td><b>name</b></td>
  <td>Hidden service name.</td>
</tr>
<tr>
  <td><b>ports</b></td>
  <td>List of <tt>HiddenServicePort</tt> values.</td>
</tr>
<tr>
  <td>authorized_clients</td>
  <td>List of client authorizations.</td>
</tr>
</table>

Example:

```yaml
tor_default_hidden_services:
  - name: ssh
    ports: [22]
```
Result:

```
HiddenServiceDir /var/lib/tor/hs_ssh/
HiddenServicePort 22
HiddenServiceVersion 3
```

### Client Authorization

You can use the included script [tor-onion-auth](scripts/tor-onion-auth) to
create v3 hidden service [authorization files](https://2019.www.torproject.org/docs/tor-manual.html.en#_client_authorization).

Example:

```bash
tor-onion-auth 1234example.onion my-client
````

Private authorization files for a Tor client are configured like this:

```yaml
tor_default_client_onion_auth: [my-client, my-other-client]
```

Public authorization files for a Tor hidden service are configured like this:

```yaml
tor_default_hidden_services:
  - name: http
    ports: [80]
    authorized_clients: [my-client, my-other-client]
```

## License

GPLv3
