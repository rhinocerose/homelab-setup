# Ansible Overview

Ansible configuration files are in YAML format. See [here](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) for more info on formatting and syntax.
## Hosts

Hosts can be defined in a host (ha!) of ways. We can define it as an `inventory.yml` file:

### Hosts Inventory file with separate group variables (`group_vars`)
```yaml
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
    east:
      hosts:
        foo.example.com:
        one.example.com:
        two.example.com:
    west:
      hosts:
        bar.example.com:
        three.example.com:
    prod:
      children:
        east:
    test:
      children:
        west:

```

If this file is not placed in `etc/...` then it must be run with the `-i {FILEPATH}` flag.

### Hosts with Address Included
```yaml
...
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
```

### Range of Hosts

```yaml
...
  webservers:
    hosts:
      www[01:50].example.com:
```
Incrementing by 2:
```yaml
...
  webservers:
    hosts:
      www[01:50:2].example.com:
```
