# Ansible Overview

Ansible configuration files are in YAML format. See [here](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) for more info on formatting and syntax.

## Setting hashed password

Run the following command:
```bash

```

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

## Variables

Order:

  - default
  - group
  
### `group_vars`
#### Directory Structure
```
├── group_vars/
│   ├── all/
│   │   ├── vars.yml
│   │   └── vault.yml
│   ├── group_name/
│   │   ├── vars.yml
│   │   └── vault.yml

```

### `host_vars`


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

### Keep vaulted variables safely visible

You should encrypt sensitive or secret variables with Ansible Vault. However, encrypting the variable names as well as the variable values makes it hard to find the source of the values. You can keep the names of your variables accessible (by grep, for example) without exposing any secrets by adding a layer of indirection:

  - Create a `group_vars/` subdirectory named after the group.
  - Inside this subdirectory, create two files named `vars` and `vault`.
  - In the `vars` file, define all of the variables needed, including any sensitive ones.
  - Copy all of the sensitive variables over to the `vault` file and prefix these variables with `vault_`.
  - Adjust the variables in the `vars` file to point to the matching `vault_` variables using `jinja2` syntax: `db_password: {{ vault_db_password }}`.
  - Encrypt the `vault` file to protect its contents.
  - Use the variable name from the `vars` file in your playbooks.
    
When running a playbook, Ansible finds the variables in the unencrypted file, which pulls the sensitive variable values from the encrypted file. 

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

## Templates
### Using Templates to create files

```yaml
- name: 'write .zshrc for users'
  template:
    src: zshrc.j2
    dest: '~{{ ansible_user_id }}/.zshrc'
    backup: yes
    mode: 'u=rw,go=r'
```
or:
```yaml
- name: config file
  template:
    src: prometheus.conf.j2
    dest: /etc/prometheus/prometheus.conf
```
### Templates that start services
```yaml
- name: Copy systemd init file
  template:
    src: init.service.j2
    dest: /etc/systemd/system/prometheus.service
  notify: systemd_reload
```


## Tasks


## Directory structure:
```
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │    └── main.yml
│   │   ├── templates/
│   │   │    └── zshrc.j2
│   │   │    └── prometheus.conf.j2
│   │   ├── vars/
│   │   │    └── main.yml
│   │   │    └── vault.yml
```
The `vars/main.yml` or `vault.yml` files could look like this:
```yaml
---
serviceName: "prometheus"
userId: "prometheus"
groupId: "prometheus"
exec_command: "/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.conf --storage.tsdb.path=/data/prometheus --storage.tsdb.retention=2d"
prometheusVersion: "2.18.1"
grafanaVersion: "7.0.0"
```

This can be used in a task like this:
```yaml
- name: 'Install Grafana'
  apt:
    deb: '/tmp/grafana_{{ grafanaVersion }}_armhf.deb'
```


### Mounting filesystem
```yaml
- name: mount disks
  mount:
    name: "{{ item.name }}"
    src: "{{ item.src }}"
    fstype: "{{ item.fs }}"
    # change to 'mounted' to auto mount
    state: present
  with_items:
    - { name: /mnt/parity1, src: /dev/disk/by-id/ata-WDC_WD60EZRZ-00GZ5B1_WD-WXN1H8449UPL-part1, fs: ext4}
    - { name: /mnt/parity2, src: /dev/disk/by-id/ata-WDC_WD80EZZX-11CSGA0_VK0U9B3Y-part1, fs: ext4}
    - { name: /mnt/disk1, src: /dev/disk/by-id/ata-WDC_WD60EZRZ-00GZ5B1_WD-WX11D55PXTV3-part1, fs: ext4}
    - { name: /mnt/retired.disk2, src: /dev/disk/by-id/ata-Hitachi_HDS5C3030ALA630_MJ1311YNG5SD3A-part1, fs: xfs}
    - { name: /mnt/disk3, src: /dev/disk/by-id/ata-WDC_WD30EFRX-68AX9N0_WD-WCC1T0632015-part1, fs: xfs}
    - { name: /mnt/disk4, src: /dev/disk/by-id/ata-TOSHIBA_DT01ACA300_X3544DGKS-part1, fs: xfs}
    - { name: /mnt/disk5, src: /dev/disk/by-id/ata-WDC_WD30EFRX-68AX9N0_WD-WMC1T0074096-part1, fs: xfs}
    - { name: /mnt/disk6, src: /dev/disk/by-id/ata-Hitachi_HDS5C3030ALA630_MJ1311YNG7SAZA-part1, fs: xfs}
```

### Start Services
```yaml
- name: Start Grafana service
  become: yes
  become_user: root
  become_method: sudo
  service:
    name: grafana-server
    state: started
    enabled: yes
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
### Generate SSH Keys

```yaml
- name: generate SSH key
  hosts: 127.0.0.1
  connection: local

  vars:
    ssh_key_filename: id_rsa_myproject

  tasks:

    - name: generate SSH key "{{ssh_key_filename}}"
      openssh_keypair:
        path: "~/.ssh/{{ssh_key_filename}}"
        type: rsa
        size: 4096
        state: present
        force: no
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

### Delete file or directory
```yaml
- name: Delete prometheus tmp folder
  file:
    path: '/tmp/prometheus-{{ prometheusVersion }}.linux-armv7'
    state: absent
```

```yaml
- name: Creates directory
  become: yes
  become_user: root
  become_method: sudo
  file: 
    path: "/data/prometheus/"
    state: directory
    owner: "{{userId}}"
    group: "{{groupId}}"
    mode: 0755
```

### Install downloaded `.deb` files
```yaml
- name: 'Install Grafana'
  apt:
    deb: '/tmp/grafana_{{ grafanaVersion }}_armhf.deb'
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

# `virtualenv` configuration thoughts
From [here](https://github.com/cchurch/ansible-role-virtualenv)
```yaml
- hosts: all
  roles:
    - name: cchurch.virtualenv
      vars:
        virtualenv_path: ~/env
        virtualenv_os_packages:
          apt: [libjpeg-dev]
          yum: [libjpeg-devel]
        virtualenv_pre_packages:
          - name: Django
            version: 1.11.26
          - Pillow
        virtualenv_requirements:
          - ~/src/requirements.txt
        virtualenv_post_packages:
          - name: PIL
            state: absent
  handlers:
    - name: custom virtualenv handler
      debug:
        msg: "virtualenv in {{ virtualenv_path }} was updated."
      listen: virtualenv updated
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

