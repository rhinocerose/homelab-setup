## Data locations

By default `docker` volumes are stored in `/var/lib/docker/volumes/{container_name}/_data/`

## Volumes

We’re going to start by creating two Docker volumes. One for Grafana’s configuration file, and another for its persistent data (like the dashboards and their layouts). Volumes are handy because they counteract the ephemeral nature of containers; when a container is destroyed but brought back up with the same volume, everything is just as you left it.
```bash
sudo docker volume create grafana-config
sudo docker volume create grafana-data
```

Then we can run:
```bash
sudo docker run \
  --name=grafana \
  --restart=always \
  --detach=true \
  --publish=3000:3000 \
  --volume=grafana-data:/var/lib/grafana \
  --volume=grafana-config:/etc/grafana \
  -e "GF_SECURITY_ADMIN_PASSWORD=hKg9szzXaQ7NkiRbT97n" \
  grafana/grafana
  ```
#### Anatomy of the command:
- `sudo docker run`: Docker needs to run as root, so we preface the command with sudo.
- `--name=grafana \`: The name of the container and how we’ll reference it in later commands.
- `--restart=always`: This flag denotes that we want the system to always reboot the container in the event of a failure, as well as automatically when Docker starts.
- `--detach=true`: We’ll want this container to run in the background in “detached” mode.
- `--publish=3000:3000`: This will publish the internal port 3000 on the host’s port 3000. This is the port that Grafana runs on, and how we’ll access the web interface.
- `--volume=grafana-data:/var/lib/grafana`: This mounts the `grafana-data` volume that we created earlier in container’s filesystem at `/var/lib/grafana`, which is defined in the `Dockerfile` as where it will look for the Grafana data.
- `--volume=grafana-config:/etc/grafana`: This mounts the `grafana-config` volume that we created earlier in the container’s filesystem at /etc/grafana, which is defined in the `Dockerfile` as where it will look for the Grafana configuration.
- `-e "GF_SECURITY_ADMIN_PASSWORD=hKg9szzXaQ7NkiRbT97n"`: This sets an environment variable that will be used by the container to set the default admin user’s password. You should change this value.
