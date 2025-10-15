# Pi-Hole
Pi-Hole installation from public release.

## Requirements
Pi-Hole hosts should be configured with static IP's per Pi-Hole documentation.

Known Issue: [list deletion via API is currently broken](https://github.com/r-pufky/ansible_pihole/issues/49).

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
Read defaults documentation. All settings not set in `pihole.toml` are
configured via PiHole's REST API when enabled.

For multiple Pi-Hole nodes apply configuration in group_vars and node specific
settings in host_vars. Singleton instances can be applied in host_vars.

Configure dual PiHole DNS servers with TOTP, REST API enable; with custom ad
and domain blocklists; backing up configurations with teleporter after all
changes are completed.

group_vars/pihole/vars/pihole.yml
``` yaml
pihole_cfg_mgmt_lists:
  - address: 'https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts'
    type: 'block'
    comment: 'Migrated from /etc/pihole/adlists.list'
    groups: ['Default']
    enabled: true

pihole_cfg_mgmt_domains:
  - domain: 'choice.microsoft.com'
    type: 'deny'
    kind: 'exact'
    comment: 'ansible blacklist'
    groups: ['Default']
    enabled: true
  - domain: 'events.gfe.nvidia.com'
    type: 'deny'
    kind: 'exact'
    comment: 'ansible blacklist'
    groups: ['Default']
    enabled: true
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
    pihole_cfg_webserver_api_totp_secret: 'CLAH6OEOV52XVYTKHGKBERP42IUZHY4D'
    pihole_cfg_webserver_api_app_pwhash: 'rest_api_test'
    pihole_srv_backup_post_enable: true
```
If multiple pihole servers are configured, it is highly recommended to use
`serial: 1`. This will apply changes to pihole server individually allowing for
changes to be applied without DNS service interruption.

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_docs/blob/main/ansible/environment.md)

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
