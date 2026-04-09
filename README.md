# [PiHole][h]
PiHole installation from public release.

## [Requirements][i]
Requires [r_pufky.srv][g] galaxy-ng collection. See
[additional documentation][m] and [reference documentation][h] for
troubleshooting and config variables.

Install size: ~30MB

> PiHole hosts should be configured with static IP's per PiHole documentation.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage
For multiple PiHole nodes apply configuration in group_vars and node specific
settings in host_vars.

### STOP
> Role has changed substantially from [5.x.x](#releases) release. Breaking
> changes were made due to breaking ansible 2.19 changes and decoupling the
> role from major PiHole releases to prevent future breaking changes.
>
> Existing role configuration must be re-evaluated, including copying existing
> **pihole.toml** to the ansible controller for templated deployments. See
> [role user defaults][j].

  Path              | Usage
 -------------------|-------
 /etc/pihole        | PiHole always deployed here.
 /etc/pihole/conf.d | pihole_conf_d always deployed here.

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                      | Notes
 ------|---------------------------|-------
  1    | pihole_flg_generate       | Generate PiHole secret hashes and exit.
  2    | pihole_flg_pre_backup     | Create backup before making changes?
  3    | pihole_flg_update         | Update pihole before configuration.
  4    | pihole_flg_api            | Enable configuration via REST API.
  5    | pihole_flg_gravity_update | Run gravity updates after configuration.
  6    | pihole_flg_post_backup    | Create backup after making changes?

### Example Playbooks

#### New Deployments (static secrets)
Almost all PiHole deployments should configure secrets and store them on the
ansible controller to create reproducible deployments.

``` yaml
# Generate TOTP, WebUI, REST API hashes to include in pihole.toml sourced file.
# Copy output and place in toml file.
- name: 'Generate PiHole secrets.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.pihole'
  vars:
    pihole_flg_generate: true
    pihole_cfg_pwhash: 'test'
    pihole_cfg_app_pwhash: 'test2'
```

A base [pihole.toml][l] file is included as a starting point for adding created
secrets to ansible sourced pihole.toml.

``` yaml
# Deploy a pre-configured PiHole instance.
- name: 'Generate PiHole secrets.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.pihole'
  vars:
    pihole_flg_api: true
    pihole_srv_restart: true
    pihole_srv_cfg: 'host_vars/pihole.example.com/pihole.toml'
    pihole_cfg_app_pwhash: 'test2'  # Required for REST API use.
```

#### New Deployments (no configuration)

``` yaml
# Create a PiHole instance with Quad9 being used for DNS servers to prevent
# external DNS logging. Passwords and TOTP secrets will be auto-generated from
# PiHole (not recommended).
- name: 'Create new PiHole instance.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.pihole'
```

#### List Management
Configure dual PiHole DNS servers with TOTP, REST API enabled; with custom ad
and domain blocklists; backing up configurations with Teleporter after all
changes are completed to the ansible controller.

``` yaml
- name: 'Deploy a pre-configured PiHole instance with custom blocklists.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.pihole'
  vars:
    pihole_flg_api: true
    pihole_srv_restart: true
    pihole_srv_cfg: 'host_vars/pihole.example.com/pihole.toml'
    pihole_cfg_app_pwhash: 'test2'  # Required for REST API use.
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

#### Multi-instance Deployments
If multiple pihole servers are configured, it is highly recommended to use
`serial: 1`. This will apply changes to pihole server individually allowing for
changes to be applied without DNS service interruption.

``` yaml
- name: 'pihole servers'
  hosts: 'pihole'
  serial: 1
  become: true
  roles:
     - 'r_pufky.srv.pihole'
  vars:
    pihole_flg_api: true
    pihole_srv_restart: true
    pihole_srv_cfg: 'host_vars/pihole.example.com/pihole.toml'
    pihole_cfg_app_pwhash: 'test2'  # Required for REST API use.
```


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
Focused on service deployment with templated configuration to minimize role
churn due to inconsistent and rapid rolling release cycle.

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
[g]: https://github.com/r-pufky/ansible_collection_srv
[h]: https://docs.PiHole.net
[i]: https://github.com/r-pufky/ansible_pihole/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_pihole/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_pihole/blob/main/defaults/main/ports.yml
[l]: https://github.com/r-pufky/ansible_pihole/blob/main/templates/default/pihole.toml
[m]: https://r-pufky.github.io/docs/service/pihole