Firewall
=========

Install and configure the firewall. For Ubuntu and RHEL/CentOS 6, an iptables template is copied and the iptables service is invoked. For RHEL/CentOS 7, the `firewalld` module is used to configure the firewall.

Role Variables
--------------

Variables defined in `defaults/main.yml`:

Set default policy to `DROP` and restrict types of ICMP traffic:

    firewall_strict: False

Set default policy to `DROP`:

    firewall_default_drop: False

Allow all ICMP traffic. Has no affect if `firewall_strict` is `True`:

    firewall_allow_icmp: True

List of TCP and UDP ports to open:

    firewall_allowed_tcp_ports:
      - 22
      - 80
      ...
    firewall_allowed_udp_ports:
      - 631
      - 123
      ...

Specify a source IP and destination port instead of opening the port globally. Optionally allow it only if it is new:

    firewall_advanced_rules:
          - source: '10.0.1.17'
            protocol: 'tcp'
            dest_port: 22
            new: True

            - source: '192.168.0.0/24'
            protocol: 'tcp'
            dest_port: 22

Rules inserted verbatim. Make sure the syntax is correct:

    firewall_custom_rules:
      - 'custom rule one'
      - 'custom rule two'


Example Playbooks
----------------

Parameterized role that restricts ICMP traffic, sets the default policy to `DROP`, passes in TCP and UDP ports to open:

```yaml
- hosts: servers
  roles:
     - { role: sdoran.firewall, firewall_strict: True, firewall_allowed_tcp_ports: [ '22' , '80' , '443'], firewall_allowed_udp_ports: [ '123' , '67' ] }
```

Use advanced rules to restrict access to services based on IP on subnet:

```yaml
- hosts: servers
  vars:
    firewall_advanced_rules:
      - source: 192.168.40.64/29
        protocol: 'tcp'
        dest_port: '22'

      - source: 192.168.41.0/24
        protocol: 'tcp'
        dest_port: '443'

  roles:
    - sdoran.firewall
```

License
-------

MIT
