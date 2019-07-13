Changelog
=========

1.10.1
------
* Change keyserver to URL for apt-key

1.10.0
------
* Remove unbound from tor_packages_exit, add tor_packages_extra
* Add global and local exit policy config for exits
* Add tor_nameservers variable
* Improve documentation

1.9.0
-----
* Add reduced_exit_policy property for exit relays

1.8.0
-----
* Use new loop keyword, remove nested loop
* Copy local RSA key
* Fix AppArmor handling

1.7.0
-----
* Update apt parameters for Ansible 2.7

1.6.0
-----
* Install deb.torproject.org-keyring
* Template RelayBandwidth* only if present

1.5.0
-----
* Configure MyFamily as multiple lines

1.4.0
-----
* Add version property for hidden services
* Add exit_addresses property for exit relays

1.3.0
-----
* Add max_mem_in_queues instance variable

1.2.0
-----
* Add tor_exit_policy_blocks variable

1.1.0
-----
* Add support for multiple DirPort values
* Add HiddenServiceAuthorizeClient support
* Add NumCPUs support

1.0.2
-----
* Copy offline keys in nested loop, add tag

1.0.1
-----
* Add ipv6_exit property

1.0.0
-----
* Initial release
