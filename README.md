# Pi-Hole
Pi-Hole installation from public release.

## Requirements
Pi-Hole hosts should be configured with static IP's per Pi-Hole documentation.

RedHat based support is **experimental** and best-effort only.

## Role Variables
Settings have been thoroughly documented for usage.

[defaults/main.yml](https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/main.yml).

[defaults/blocklist.yml](https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/blocklist.yml).

[defaults/ftl.yml](https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/ftl.yml).

### Ports
All ports and protocols have been defined for the role.

Hosts should only define firewall rules for ports they need.

[defaults/ports.yml](https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/ports.yml).

Redhat based installs will create a ``pihole`` zone in ``firewalld`` and allow
traffic through.

## Dependencies
N/A

## Example Playbook
For multiple Pi-Hole nodes apply configuration in group_vars and node specific
settings in host_vars. Singleton instances can be applied in host_vars.

group_vars/pihole/vars/pihole.yml
``` yaml
pihole_webpassword: '{{ vault_pihole_webpassword }}'

pihole_ad_sources:
  - id: 1
    address: 'https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts'
    enabled: true
    comment: 'Migrated from /etc/pihole/adlists.list'
  - id: 2
    address: 'https://adaway.org/hosts.txt'
    enabled: true
    comment: 'ansible adlist'
  - id: 3
    address: 'https://bitbucket.org/ethanr/dns-blacklists/raw/8575c9f96e5b4a1308f2f12394abd86d0927a4a0/bad_lists/Mandiant_APT1_Report_Appendix_D.txt'
    enabled: true
    comment: 'ansible adlist'

pihole_domain_blocklists:
  - id: 1
    type: 1
    domain: 'choice.microsoft.com'
    enabled: true
    comment: 'ansible blacklist'
  - id: 2
    type: 1
    domain: 'events.gfe.nvidia.com'
    enabled: true
    comment: 'ansible blacklist'
```

host_vars/pihole.example.com/vars/pihole.yml
``` yaml
pihole_pihole_interface: 'eth0'
pihole_ipv4_address:     '10.9.9.2/24'
pihole_ipv6_address:     ''
pihole_pihole_dns_1:     '10.9.9.1#53'
pihole_pihole_dns_2:     ''
```

host_vars/pihole2.example.com/vars/pihole.yml
``` yaml
pihole_pihole_interface: 'eth0'
pihole_ipv4_address:     '10.9.9.3/24'
pihole_ipv6_address:     ''
pihole_pihole_dns_1:     '10.9.9.1#53'
pihole_pihole_dns_2:     ''
```

site.yml
``` yaml
- name:   'pihole servers'
  hosts:  'pihole'
  become: true
  roles:
     - 'r_pufky.pihole'
```
If multiple pihole servers are configured, it is highly recommended to use
`serial: 1`. This will apply changes to pihole server individually allowing for
changes to be applied without DNS service interruption.

## Versions

**3.x: FTL Configuration Support**
* Add FTL-DNS configuration support.
* Add idempotent operational toggle.
* Standardize setupvars to YAML datatypes (no existing change required).
* Enable management of default adlist.
* Document undocumented 'setupvars.conf' options.

Consumers who have set custom FTL settings should ensure they have set these in
*_vars before applying this version. See:

[defaults/ftl.yml](https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/ftl.yml).

**2.x: RedHat Support**
* Redhat support. This is best-effort support only.
* Conditional forwarding configuration support.
* Added ports.yml usage reference for data consumption.

**1.x: Initial Release**
* Add support for updating pihole installation.
* Add DHCP configuration, CLI domain list management.
* Allow running in check_mode.
* Reconfigure pihole on configuration change (opposed to restart).
* Support for pihole CLI domain whitelist/blacklist management.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

RedHat support is best-effort only, and should be assigned to @rkoosaar.

## License
[AGPL-3.0 License](https://github.com/r-pufky/ansible_pihole/blob/main/LICENSE)

## Author Information
https://keybase.io/rpufky
