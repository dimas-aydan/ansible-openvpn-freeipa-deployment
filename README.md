# Integrate OpenVPN and FreeIPA (Including FreeIPA Replica) Using Ansible.

Notes :
1. This project has been tested on AlmaLinux 10. Other RHEL 10–compatible distributions should also work, but they have not been explicitly tested.
2. The FreeIPA server is used exclusively for OpenVPN user authentication. As a result, FreeIPA DNS, CA, NTP, and other optional FreeIPA services are not deployed in this project.
3. Because FreeIPA DNS is not configured, you must manually create the required DNS records on your own DNS server (or manage name resolution via /etc/hosts).
4. A FreeIPA Replica is included in this project to provide high availability for authentication services.

Prerequisite :
1. Create DNS SRV records for the FreeIPA Server and FreeIPA Replica

```
_ldap._tcp	SRV	100 389 vmX.example.com 0
_kerberos._tcp	SRV	100 88 vmX.example.com 0
_kerberos._udp	SRV	100 88 vmX.example.com 0
```

You may adjust the **priority** and **weight** values based on your environment and failover design.

Proper DNS SRV records are mandatory for Kerberos authentication and FreeIPA high availability.
Incorrect or missing records may cause authentication failures during server failover.

2. Configure SSH key-based authentication
- Generate an SSH key on the Ansible control node (if not already present)
- Ensure the Ansible control node can access all target machines using SSH key-based authentication
- Password-based SSH access is not recommended

3. Rename the example inventory file:

```
inventories/production/hosts_example.yml → hosts.yml
```

4. Rename the example variable file:

```
inventories/production/group_vars/all_example.yml → all.yml
```

5. Use a fresh operating system installation (Recommended)

Project Structure :
```
ansible
│   .gitignore
│   ansible.cfg
│   README.md
│
├───client
│       client.conf.j2
│       client_generate.yml
│
├───inventories
│   └───production
│       │   hosts.yml
│       │   hosts_example.yml
│       │
│       └───group_vars
│               all.yml
│               all_example.yml
│               openvpn.yml
│
├───playbooks
│       freeipa.yml
│       openvpn.yml
│       replica.yml
│       site.yml
│
└───roles
    ├───common
    │   └───tasks
    │           main.yml
    │
    ├───firewall
    │   ├───handlers
    │   │       main.yml
    │   │
    │   ├───tasks
    │   │       main.yml
    │   │       setup.yml
    │   │       systemd.yml
    │   │
    │   └───templates
    │           add_bridge.sh.j2
    │           override.conf.j2
    │           remove_bridge.sh.j2
    │
    ├───freeipa_client
    │   └───tasks
    │           enroll.yml
    │           install.yml
    │           main.yml
    │
    ├───freeipa_replica
    │   └───tasks
    │           main.yml
    │
    ├───freeipa_server
    │   └───tasks
    │           configure.yml
    │           install.yml
    │           main.yml
    │
    ├───hardening
    │   └───tasks
    │           main.yml
    │
    └───openvpn_server
        ├───handlers
        │       main.yml
        │
        ├───tasks
        │       configure.yml
        │       install.yml
        │       main.yml
        │       pam_sssd.yml
        │       pki.yml
        │
        └───templates
                openvpn_pam.j2
                server.conf.j2
```

# About Client Folder & Playbook
The playbook client/client_generate.yml is used to generate OpenVPN Client config (.ovpn/.conf) and create the user in FreeIPA

How to Run :
```
ansible-playbook client/client_generate.yml -e user_pass="your_password" -e username="your_name"
```
