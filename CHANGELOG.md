# Changelog

## 2.15.0

- Remove `tor_apparmor` variable
- Add `tor_systemd_new_privileges` variable

## 2.14.0

- Update Tor keyring checksum

## 2.13.0

- Remove explicit `ExitPolicy` for bridges
- Add pluggable transport `options` for bridges

## 2.12.0

- Update Tor signing key handling

## 2.11.0

- Add systemd restart/timeout variables
- Add `tor_instances_remove` variable

## 2.10.0

- Add `tor_default_settings` variable
- Include `/etc/tor/exit-policy-blocks*`

## 2.9.0

- Remove `max_advertised_bandwidth` instance variable
- Remove `relay_bandwidth_rate` instance variable
- Remove `relay_bandwidth_burst` instance variable
- Add `extra_settings` instance variable

## 2.8.1

- Add missing `copy-offline-keys` tag

## 2.8.0

- Backup new RSA keys
- Add `max_advertised_bandwidth` instance variable

## 2.7.1

- Reformat `torrc` templates for better Ansible compatibility

## 2.7.0

- Remove `tor_exit_notice` and `dir_ports` variables
- Add `MetricsPort` variables

## 2.6.0

- Remove per-instance `contact_info` variable
- Add `ext_or_port` for bridges
- Add `tor_repository` variable
- Add tags for skipping certain tasks

## 2.5.0

- Remove support for v2 Hidden Services
- Remove `exit_addresses` instance variable
- Add `address` instance variable
- Add `tor_contact_info` variable

## 2.4.0

- Add `outbound_addresses` instance variable
- Remove copying `exit-policy-blocks`

## 2.3.0

- Add v3 HS client authorization properties

## 2.2.0

- Add offline key and v3 HS auth creation scripts

## 2.1.0

- Copy and auto-include local `exit-policy-blocks`

## 2.0.0

- Replace some torrc variables with generic property
- Remove instance-specific exit policy properties
- Cleanup various files
- Reorganize template files
- Remove relay type-specific packages variables

## 1.10.3

- Make sure GnuPG is installed
- Copy Tor exit notice only to exits

## 1.10.2

- Fix Ansible deprecation warnings

## 1.10.1

- Change keyserver to URL for apt-key

## 1.10.0

- Remove unbound from `tor_packages_exit`, add `tor_packages_extra`
- Add global and local exit policy config for exits
- Add `tor_nameservers` variable
- Improve documentation

## 1.9.0

- Add `reduced_exit_policy` property for exit relays

## 1.8.0

- Use new `loop` keyword, remove nested loop
- Copy local RSA key
- Fix AppArmor handling

## 1.7.0

- Update apt parameters for Ansible 2.7

## 1.6.0

- Install `deb.torproject.org-keyring`
- Template `RelayBandwidth-` only if present

## 1.5.0

- Configure `MyFamily` as multiple lines

## 1.4.0

- Add `version` property for hidden services
- Add `exit_addresses` property for exit relays

## 1.3.0

- Add `max_mem_in_queues` instance variable

## 1.2.0

- Add `tor_exit_policy_blocks` variable

## 1.1.0

- Add support for multiple `DirPort` values
- Add `HiddenServiceAuthorizeClient` support
- Add `NumCPUs` support

## 1.0.2

- Copy offline keys in nested loop, add tag

## 1.0.1

- Add `ipv6_exit` property

## 1.0.0

- Initial release
