:warning:

Migration to ansible 2.18.x in progress. There will be substantial changes to
the role comply with new Galaxy and Collection requirements.

[3.x](https://github.com/r-pufky/ansible_pihole/tree/3.x) is the last version
pre ansible 2.18. Anyone wishing to use the old version and supported platforms
should use this branch. **3.x** will **no longer** be actively maintained.

* Open issues will be addressed during migration or resolved if no longer
  relevant.
* Experimentally supported OS's will be removed due to lack of contributions.
* Testing standardized with implementations used across all other existing
  roles/collections.
* Documentation standardized and updated.
* Jump to latest PiHole version.

# Pi-Hole
Pi-Hole installation from public release.

## Requirements
Pi-Hole hosts should be configured with static IP's per Pi-Hole documentation.

[supported platforms](https://github.com/r-pufky/ansible_pihole/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_pihole/tree/main/defaults/main)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/ports.yml).

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv) collection.

## Example Playbook
Read defaults documentation.

For multiple Pi-Hole nodes apply configuration in group_vars and node specific
settings in host_vars. Singleton instances can be applied in host_vars.

Configure dual PiHole DNS servers with TOTP, REST API enable; with custom ad
and domain blocklists.

group_vars/pihole/vars/pihole.yml
``` yaml
pihole_webpassword: '{{ vault_pihole_webpassword }}'

pihole_cfg_mgmt_ad_sources:
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

pihole_cfg_mgmt_domain_blocklists:
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
pihole_cfg_dns_host_force4: true
pihole_cfg_dns_host_ipv4: '10.9.9.2'
pihole_cfg_dns_upstreams:
  - '10.9.9.1#53'
  - '8.8.8.8'
  - '4.4.4.4'
```

host_vars/pihole2.example.com/vars/pihole.yml
``` yaml
pihole_cfg_dns_host_force4: true
pihole_cfg_dns_host_ipv4: '10.9.9.3'
pihole_cfg_dns_upstreams:
  - '10.9.9.1#53'
  - '8.8.8.8'
  - '4.4.4.4'
```

site.yml
``` yaml
- name: 'pihole servers'
  hosts: 'pihole'
  serial: 1
  become: true
  roles:
     - 'r_pufky.srv.pihole'
  vars:
    pihole_cfg_webserver_api_pwhash: 'test'
    pihole_cfg_webserver_api_totp_secret:
      'CLAH6OEOV52XVYTKHGKBERP42IUZHY4D'  # Well known TOTP example secret.
    pihole_cfg_webserver_api_app_pwhash: 'rest_api_test'
```
If multiple pihole servers are configured, it is highly recommended to use
`serial: 1`. This will apply changes to pihole server individually allowing for
changes to be applied without DNS service interruption.

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
Release format: **{OS}-{SERVICE}-{ROLE}**

Each type inherits the versioning system used; defaulting to schematic
versioning.

`12.0.0-2.0.3-1.0.0`

* 12.0.0 - Debian 12 (bookworm).
* 2.0.3 - Service/app version.
* 1.0.0 - Role version.

Releases are branched on Debian releases:

* **[13.x.x](https://github.com/r-pufky/ansible_pihole)**: 13 Trixie.
* **[3.x](https://github.com/r-pufky/ansible_pihole/tree/3.x)**: Legacy role
  PiHole 3.x, Ansible 2.11.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_deluge/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
