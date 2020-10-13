# Ansible Overview

Ansible configuration files are in YAML format. See [here](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) for more info on formatting and syntax.
## Hosts

Hosts can be defined in a host (ha!) of ways. We can define it as an `inventory.yml` file:

```yaml

```

If this file is not placed in `etc/...` then it must be run with the `-i {FILEPATH}` flag.

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
