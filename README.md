Firewall
=========
[![Galaxy](https://img.shields.io/badge/galaxy-samdoran.firewall-blue.svg?style=flat)](https://galaxy.ansible.com/samdoran/firewall)
[![Build Status](https://dev.azure.com/samdoran/ansible-role-firewall/_apis/build/status/samdoran.ansible-role-firewall?branchName=main)](https://dev.azure.com/samdoran/ansible-role-firewall/_build/latest?definitionId=1&branchName=main)

This role will install and configure the firewall. It supports `iptables`, `firewalld`, and `pf`.

For Ubuntu and RHEL/CentOS 6, an `iptables` template is copied and the iptables service is invoked. For RHEL/CentOS 7 or later, the `firewalld` module is used to configure the firewall.

For macOS, a `pf.conf` template is used. The [application firewall][_alf] can be disabled or enabled. If enabled, stealth mode will be disabled since the `pf.conf` template manages the same rules set my stealth mode.

`firewalld` rules are currently _additive_ and will not "clean up" the firewall on a running system.

Dependencies
------------

- `ansible.posix` collection

Role Variables
--------------

These variables apply to all firewall types:

|   Name               | Default Value | Description                                                      |
|----------------------|---------------|------------------------------------------------------------------|
| `firewall_strict`    | `false`         | Set default policy to `DROP` and restrict types of ICMP traffic  |
| `firewall_default_drop`    | `false`  | Set default policy to `DROP`  |
| `firewall_allow_icmp` | `true` | Allow all ICMP traffic. Has no affect if `firewall_strict` is `True` |
| `firewall_allowed_tcp_ports` | `['22']` | List of allowed TCP ports |
| `firewall_allowed_udp_ports` | `['161'] `| List of allowed UDP ports |
| `firewall_rich_rules` | `[]` | Specify a source IP and destination port instead of opening the port globally. Optionally allow it only if it is new. With `iptables`, this adds rules to the `iptables` config file. With `firewalld`, this creates rich rules to the specified zone. |
| `firewall_custom_pf_rules` | `[]` | Rules insterted verbatim for `pf` at the end of all other filter rules. Make sure the syntax is correct. |
| `firewall_nat_rules` | `[]` | List of ports and their protocols to NAT. With `iptables`, add prerouting rules to the `NAT` table. With `firewalld`, adds rich rules to the specified zone. |


`firewalld` specific variables:

|   Name               | Default Value | Description                                                      |
|----------------------|---------------|------------------------------------------------------------------|
| `firewall_firewalld_rules` | `[]` | List of rules to pass to the `firewalld` module. Each module argument is optional. |

`iptables` specific variables:

|   Name               | Default Value | Description                                                      |
|----------------------|---------------|------------------------------------------------------------------|
| `firewall_custom_iptables_rules` | `[]` | Rules inserted verbatim for `iptables`. Make sure the syntax is correct. |

macOS (`pf`) and [application firewall][_alf] specific variables. The custom rules are inserted in the appropciate section in the `pf.conf` template. The syntax must be correct or valiadtion will fail. See the man page of [`pf.conf`](https://man.openbsd.org/pf.conf.5) for details.:

|   Name               | Default Value | Description                                                      |
|----------------------|---------------|------------------------------------------------------------------|
| `firewall_alf_global_state` | `on` | State of the macOS [application firewall][_alf]. |
| `firewall_custom_pf_macros` | `[]` | Custom `pf` macros. |
| `firewall_custom_pf_table_rules` | `[]` | Custom `pf` table rules. |
| `firewall_custom_pf_options` | `[]` | Custom `pf` options. |
| `firewall_custom_pf_normalization_rules` | `[]` | Custom `pf` normalation rules. |
| `firewall_custom_pf_queuing_rules` | `[]` | Custom `pf` queuing rules. |
| `firewall_custom_pf_filter_rules` | `[]` | Custom `pf` filter rules. |

Examples:

    firewall_nat_rules
      - protocol: tcp
        original_port: 4022
        translated_port: 22

    firewall_rich_rules:
      - source: '10.0.1.17'
        protocol: 'tcp'
        dest_port: 22
        new: true

      - source: '192.168.0.0/24'
        protocol: 'tcp'
        dest_port: 22

    firewall_custom_iptables_rules:
      - 'custom rule one'
      - 'custom rule two'

    firewall_firewalld_rules:
      - port: 4444
        protocol: tcp

      - service: ceph
        timeout: 99

    firewall_custom_pf_rules:
      -


Example Playbooks
----------------

Restricts ICMP traffic, sets the default policy to `DROP`, passes in TCP and UDP ports to open:

```yaml
- hosts: all
  roles:
    - role: sdoran.firewall
      vars:
        firewall_strict: true
        firewall_allowed_tcp_ports:
          - 22
          - 80
          - 443
        firewall_allowed_udp_ports:
          - 123
          - 67
```

Use advanced rules to restrict access to services based on IP on subnet:

```yaml
- hosts: all
  vars:
    firewall_rich_rules:
      - source: 192.168.40.64/29
        protocol: 'tcp'
        dest_port: '22'

      - source: 192.168.41.0/24
        protocol: 'tcp'
        dest_port: '443'

  roles:
    - sdoran.firewall
```

NAT port to a service not running as root on a port <1024:

```yaml
- hosts: all
  vars:
    firewall_nat_rules:
      - protocol: tcp
        original_port: 514
        translated_port: 5599
```

Use a custom port for SSH and block the default port.

```yaml
- hosts: all
  vars:
    firewall_allowed_tcp_ports: []
    firewall_nat_rules:
      - protocol: tcp
        original_part: 2022
        translated_port: 22
```

License
-------

Apache 2.0


[_alf]: https://support.apple.com/en-us/HT201642
