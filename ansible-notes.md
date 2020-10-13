# Ansible Overview

Ansible configuration files are in YAML format. See [here](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) for more info on formatting and syntax.

## Directory Structure

```
├── roles/
│   ├── logging/
│   │   ├── tasks/
│   │   │   ├── setup.yml
│   │   │   ├── filebeat.yml
│   │   │   ├── elasticsearch.yml
│   │   │   ├── kibana.yml
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │    └── main.yml
│   │   ├── vars/
│   │   │    └── vault.yml
│   │   ├── files/
│   │   │    └── filebeat.yml
```

## Secrets and `ansible-vault`

Create a new secret in `vars/vault` by using:

``` bash
ansible-vault create vault.yaml
```

To automatically encrypt on exiting, use:
``` bash
ansible-vault create vault.yaml
```

To manually encrypt and decrypt, use:
``` bash
ansible-vault decrypt vault.yaml
ansible-vault encrypt vault.yaml
```

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

If this file is not placed in `/etc/ansible/hosts` then it must be run with the `-i {FILEPATH}` flag.

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

## Tasks

### Using Templates to create files

```yaml
- name: 'write .zshrc for users'
  template:
    src: zshrc.j2
    dest: '~{{ ansible_user_id }}/.zshrc'
    backup: yes
    mode: 'u=rw,go=r'
```

Directory structure:
```
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │    └── main.yml
│   │   ├── templates/
│   │   │    └── zshrc.j2
```

### Install Packages
Replace `apt` with `pacman` for Arch
```yaml
- name: 'Update and upgrade apt packages'
  become: yes
  apt:
    upgrade: yes
    update_cache: yes
    cache_valid_time: 86400
- name: 'Install packages'
  become: yes
  apt:
    pkg:
      - python3-pip
      - git
      - vim
      - zsh
      - tmux
```
### `pip` Packages

```yaml
- pip:
    name: glances
    executable: pip3
```

### Set shell
```yaml
- name: 'set default shell for users'
  become: yes
  user:
    name: pi
    shell: /bin/zsh
```

### Set file permissions
```yaml
- name: 'set permissions of oh-my-zsh for users'
  file:
    path: '~{{ ansible_user_id }}/.oh-my-zsh'
    # Prevent the cloned repository from having insecure permissions. Failing to do
    # so causes compinit() calls to fail with "command not found: compdef" errors
    # for users with insecure umasks (e.g., "002", allowing group writability).
    mode: 'go-w'
    recurse: yes
```
## Roles

## Playbooks

```yaml
---
- name: update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: ensure postgresql is at the latest version
    yum:
      name: postgresql
      state: latest
  - name: ensure that postgresql is started
    service:
      name: postgresql
      state: started
```

## Pulling Playbooks

From [Ansible examples Github](https://github.com/ansible/ansible-examples/blob/master/language_features/ansible_pull.yml):
```yaml
# ansible-pull setup
#
# on remote hosts, set up ansible to run periodically using the latest code
# from a particular checkout, in pull based fashion, inverting Ansible's
# usual push-based operating mode.
#
# This particular pull based mode is ideal for:
#
# (A) massive scale out
# (B) continual system remediation
#
# DO NOT RUN THIS AGAINST YOUR HOSTS WITHOUT CHANGING THE repo_url
# TO SOMETHING YOU HAVE PERSONALLY VERIFIED
#
#
---

- hosts: pull_mode_hosts
  remote_user: root

  vars:

    # schedule is fed directly to cron
    schedule: '*/15 * * * *'

    # User to run ansible-pull as from cron
    cron_user: root

    # File that ansible will use for logs
    logfile: /var/log/ansible-pull.log

    # Directory to where repository will be cloned
    workdir: /var/lib/ansible/local

    # Repository to check out -- YOU MUST CHANGE THIS
    # repo must contain a local.yml file at top level
    #repo_url: git://github.com/sfromm/ansible-playbooks.git
    repo_url: SUPPLY_YOUR_OWN_GIT_URL_HERE

  tasks:

    - name: Install ansible
      yum: pkg=ansible state=installed

    - name: Create local directory to work from
      file: path={{workdir}} state=directory owner=root group=root mode=0751

    - name: Copy ansible inventory file to client
      copy: src=/etc/ansible/hosts dest=/etc/ansible/hosts
              owner=root group=root mode=0644

    - name: Create crontab entry to clone/pull git repository
      template: src=templates/etc_cron.d_ansible-pull.j2 dest=/etc/cron.d/ansible-pull owner=root group=root mode=0644

    - name: Create logrotate entry for ansible-pull.log
      template: src=templates/etc_logrotate.d_ansible-pull.j2 dest=/etc/logrotate.d/ansible-pull owner=root group=root mode=0644
```

# Pi cluster example

Sourced from [here](https://www.dinofizzotti.com/blog/2020-04-10-raspberry-pi-cluster-part-1-provisioning-with-ansible-and-temperature-monitoring-using-prometheus-and-grafana/)

Inventory file `inventory.yml`
```yaml
all:
  hosts:
    whitepi:
      ansible_host: whitepi.local
    greenpi:
      ansible_host: greenpi.local
    redpi:
      ansible_host: redpi.local
    bluepi:
      ansible_host: bluepi.local
  children:
    raspberry_pi:
      hosts:
        whitepi: {}
        greenpi: {}
        redpi: {}
        bluepi: {}
    monitoring_server:
      hosts:
        whitepi: {}
  vars:
    ansible_python_interpreter: /usr/bin/python3
    remote_user: pi
```

Main playbook `up.yml`
```yaml
---
- hosts: raspberry_pi
  user: pi
  become: yes
  become_user: root
  become_method: sudo
  roles:
    - common
    - rpi_exporter
    - node_exporter

- hosts: monitoring_server 
  user: pi
  become: yes
  become_user: root
  become_method: sudo
  roles:
    - monitoring_server 
```

