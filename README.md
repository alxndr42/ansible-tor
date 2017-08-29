ansible-tor
===========

Installs Tor relays on Debian-based systems.

This role uses the default Tor instance (configuration in _/etc/tor/torrc_)
only for hidden services and optionally a SOCKS proxy. Bridge, middle and exit
relays are created as additional instances using the _tor-instance-create_
script (configuration under _/etc/tor/instances_).

Requirements
------------

* Debian-based system with systemd
* Offline keys as described below

Role Variables
--------------

Please see [defaults/main.yml](defaults/main.yml) for default values.

<table>
<tr>
  <th>Variable</th>
  <th>Description</th>
</tr>
<tr>
  <td>tor_apparmor</td>
  <td>
    Disable AppArmor, in case Tor is running in an LXC container.
    (<a href="https://trac.torproject.org/projects/tor/ticket/17754">Bug 17754</a>)
  </td>
</tr>
<tr>
  <td>tor_avoid_disk</td>
  <td>Set <tt>AvoidDiskWrites</tt>, for flash-based systems. </td>
</tr>
<tr>
  <td>tor_default_hidden_services</td>
  <td>Hidden service definitions (see below).</td>
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
  <td>tor_exit_notice</td>
  <td>Local HTML file used as <tt>DirPortFrontPage</tt> on exit instances.</td>
</tr>
<tr>
  <td>tor_instances_bridge</td>
  <td>Bridge instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_instances_exit</td>
  <td>Exit instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_instances_middle</td>
  <td>Middle instance definitions (see below).</td>
</tr>
<tr>
  <td>tor_my_family</td>
  <td>List of fingerprints for <tt>MyFamily</tt>.</td>
</tr>
<tr>
  <td>tor_num_cpus</td>
  <td><tt>NumCPUs</tt> value.</td>
</tr>
<tr>
  <td>tor_offline_keys</td>
  <td>Base directory for offline keys (see below).</td>
</tr>
<tr>
  <td>tor_packages_bridge</td>
  <td>Extra packages to install for bridge instances.</td>
</tr>
<tr>
  <td>tor_packages_exit</td>
  <td>Extra packages to install for exit instances.</td>
</tr>
<tr>
  <td>tor_packages_middle</td>
  <td>Extra packages to install for middle instances.</td>
</tr>
</table>

Offline Keys
------------

A given host's and instance's offline keys are copied from
`{{ tor_offline_keys }}/{{ inventory_hostname }}/INSTANCE_NAME/keys`.
See `man tor` for information on how to manage offline keys.

Example:

Using the default `tor_offline_keys` value, an inventory hostname of `tor-exit`
and an instance name of `exit`, offline keys would be generated with this
command:

    tor --DataDirectory ~/.tor_offline_keys/tor-exit/exit --keygen

Hidden Services
---------------

Hidden services are only configured on the default instance via the
`tor_default_hidden_services` variable.

Example:

    tor_default_hidden_services:
      - name: ssh
        authorize_client: basic ssh
        ports: [ 22 ]

Result:

    HiddenServiceDir /var/lib/tor/hs_ssh/
    HiddenServiceAuthorizeClient basic ssh
    HiddenServicePort 22

The property `authorize_client` is optional.

Instance Configuration
----------------------

Bridge, middle and exit relays are created using the _tor-instance-create_
script and configured via three separate variables in the inventory (see above).

The following properties are common to all instance types:

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td>name</td>
  <td>
    Instance name (required). Example: the instance name <tt>foo</tt> will create
    the systemd unit <tt>tor@foo</tt> owned by user/group <tt>_tor-foo</tt>.
    The configuration will be in <i>/etc/tor/instances/foo/torrc</i>,
    the data in <i>/var/lib/tor-instances/foo</i>.
  </td>
</tr>
<tr>
  <td>or_ports</td>
  <td>List of <tt>ORPort</tt> values (required).</td>
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
<tr>
  <td>contact_info</td>
  <td><tt>ContactInfo</tt> value.</td>
</tr>
</table>

### Bridge Configuration ###

The following properties are used by bridge instances:

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td>pluggable_transports</td>
  <td>List of pluggable transport definitions (see below).</td>
</tr>
</table>

Example:

    tor_instances_bridge:
      - name: bridge
        nickname: MyCoolBridge
        or_ports: [ 9000 ]
        contact_info: "Foo <foo@example.com>"
        pluggable_transports:
          - name: obfs4
            exec: /usr/bin/obfs4proxy
            address: 0.0.0.0:8443

### Middle/Exit Configuration ###

The following properties are used by middle/exit instances:

<table>
<tr>
  <th>Property</th>
  <th>Description</th>
</tr>
<tr>
  <td>dir_ports</td>
  <td>List of <tt>DirPort</tt> values (only one can be advertised).</td>
</tr>
<tr>
  <td>ipv6_exit</td>
  <td>Allow IPv6 exit traffic (exits only).</td>
</tr>
<tr>
  <td>exit_policy</td>
  <td>List of <tt>ExitPolicy</tt> values (exits only).</td>
</tr>
</table>

Example:

    tor_instances_exit:
      - name: exit
        nickname: MyCoolExit
        contact_info: "Foo <foo@example.com>"
        or_ports:
          - 443
          - "[abcd::1:2:3:4]:443"
        dir_ports:
          - 80
          - "[abcd::1:2:3:4]:80 NoAdvertise"
        ipv6_exit: 1
        exit_policy:
          - "accept *:80"
          - "accept *:443"
          [...]
          - "reject *:*"

License
-------

GPLv3
