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

[Additional documentation](http://r-pufky.github.io/docs/service/pihole).

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
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

Testing variables:

  Variable            | type | Description
 ---------------------|------|-------------
  molecule_flg_inject | bool | Flag to inject files locally.

### [Releases][b]

  Release | Debian | Ansible | PiHole  | Notes
 ---------|--------|---------|---------|-------
  6.x.x   | 13     | 2.20    | v6.4.1  | Ansible 2.20, feature flags, and semantic versioning.
  5.x.x   | 13     | 2.18    | v6.3.2  | Update to PiHole 6.3.2.
  4.x.x   | 13     | 2.18    | v6.1.4  | Migrate to Debian Trixie.
  3.x.x   | 12     | 2.18    | v11.0.3 | FTL-DNS configuration support.
  2.x.x   | 12     | 2.12    | v5.17   | Redhat (experimental), conditional forwarding support.
  1.x.x   | 12     | 2.12    | v5.8.1  | Migrate from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]


[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_pihole/blob/main/LICENSE
