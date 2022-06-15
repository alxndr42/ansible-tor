# ansible-tor

Installs Tor relays on Debian-based systems.

This role uses the default Tor instance (configuration in */etc/tor/torrc*)
for hidden services and optionally a SOCKS proxy. Bridge, middle and exit
relays are created as additional instances using the *tor-instance-create*
script (configuration under */etc/tor/instances*).

## Requirements

- Debian-based system with systemd
- Offline keys as described below

## Role Variables

Please see [defaults/main.yml](defaults/main.yml) for default values.

### Main Variables

| Variable | Description |
| --- | --- |
| `tor_contact_info` | `ContactInfo` value. |
| `tor_instances_bridge` | Bridge instance definitions (see below). |
| `tor_instances_middle` | Middle instance definitions (see below). |
| `tor_instances_exit` | Exit instance definitions (see below). |
| `tor_offline_keys` | Local base directory for offline keys (see below). |

### Other Variables

| Variable | Description |
| --- | --- |
| `tor_apparmor` | Use `false` to disable AppArmor, in case Tor is running in an LXC container. ([Bug 17754](https://gitlab.torproject.org/tpo/core/tor/-/issues/17754)) |
| `tor_avoid_disk` | Set `AvoidDiskWrites`, for flash-based systems. |
| `tor_default_client_onion_auth` | Client authorizations for the default instance (see below). |
| `tor_default_hidden_services` | Hidden service definitions for the default instance (see below). |
| `tor_default_socks_port` | SOCKS port of the default instance. |
| `tor_dist` | Distribution name on [deb.torproject.org](https://deb.torproject.org/torproject.org/dists/). |
| `tor_exit_policy` | List of `ExitPolicy` values (exits only). Overrides `tor_reduced_exit_policy`. |
| `tor_instance_settings` | List of additional settings for bridge, middle and exit instances. |
| `tor_my_family` | List of fingerprints for `MyFamily`. |
| `tor_nameservers` | List of nameserver addresses to add to */etc/tor/resolv.conf*, which will be used by bridge, middle and exit instances as `ServerDNSResolvConfFile`. |
| `tor_onion_auth` | Local directory for client authorization files (see below). |
| `tor_packages_extra` | Extra system packages to install. |
| `tor_reduced_exit_policy` | `ReducedExitPolicy` value (exits only). |
| `tor_repository` | Use the Tor Project package repository. |

## Instance Configuration

Bridge, middle and exit relays are created using the *tor-instance-create*
script and configured via three separate variables in the inventory (see above).

The following properties are common to all instance types (required properties
in **bold**):

| Property | Description |
| --- | --- |
| **`name`** | Instance name. Example: the instance name `foo` will create the systemd unit `tor@foo` owned by user/group `_tor-foo`. The configuration will be in */etc/tor/instances/foo/torrc*, the data in */var/lib/tor-instances/foo*. |
| **`or_ports`** | List of `ORPort` values. |
| `metrics_port` | `MetricsPort` value. |
| `metrics_port_policy` | `MetricsPortPolicy` value. |
| `nickname` | Relay nickname. |
| `relay_bandwidth_rate` | `RelayBandwidthRate` value. |
| `relay_bandwidth_burst` | `RelayBandwidthBurst` value. |

### Bridge Configuration

The following properties are used by bridge instances (required properties in
**bold**):

| Property | Description |
| --- | --- |
| `ext_or_port` | `ExtORPort` value. |
| **`pluggable_transports`** | List of pluggable transport definitions (see below). |

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

| Property | Description |
| --- | --- |
| `address` | The IPv4 address of this server. |
| `ipv6_exit` | Allow IPv6 exit traffic (exits only). |
| `outbound_addresses` | List of `OutboundBindAddress` values. |

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

To skip copying the offline keys, use `--skip-tags copy-offline-keys` when
applying the role.

**Please note:** Tor creates the RSA key `secret_id_key` on new relays. This
key is part of the relay identity, so you should copy it to the local directory
mentioned above. The role does this once the RSA key exists.

## Hidden Services

Hidden services are only configured on the default instance via the
`tor_default_hidden_services` variable.

The following properties are used by hidden services (required properties in
**bold**):

| Property | Description |
| --- | --- |
| **`name`** | Hidden service name. |
| **`ports`** | List of `HiddenServicePort` values. |
| `authorized_clients` | List of client authorizations. |

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

GNU General Public License v3 or later (GPLv3+)
