# Environment variables

## Filesystem Method

From [here](https://docs.organizr.app/books/tutorials/page/static-ip-ssh-environment-variables-mounting-drives-permissions-and-directories%28part-2%29)

Type in `id` to find `PUID` and `PGID`. Type in `sudo nano /etc/environment` and then:

```bash
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
PUID=1000
PGID=1000
TZ="America/New_York"
USERDIR="/home/mediaserver"
MYSQL_ROOT_PASSWORD="123456789"
```
## Ansible method

Have `{HOSTNAME}` as a role, ie `arch` or `epsilon`. In `{HOSTNAME}/files/etc/environment` store the variables.

# `fstab`

## Ansible method

Have `{HOSTNAME}` as a role, ie `arch` or `epsilon`. In `{HOSTNAME}/files/etc/fstab` store the variables.

[Example file](https://github.com/rhinocerose/ansible/blob/master/roles/epsilon/files/etc/fstab)
