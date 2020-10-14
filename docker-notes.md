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
