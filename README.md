# dockerd-exporter

Prometheus Docker daemon metrics exporter

### Docker Engine 

Create or edit /etc/systemd/system/docker.service.d/docker.conf, 
enable the experimental feature and set the metrics address to 0.0.0.0:

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// \
  --storage-driver=overlay2 \
  --dns 8.8.4.4 --dns 8.8.8.8 \
  --log-driver json-file \
  --log-opt max-size=50m --log-opt max-file=10 \
  --experimental=true \
  --metrics-addr 0.0.0.0:9323
```

Check if the docker_gwbridge ip address is `172.18.0.1`:

```bash
 ip -o addr show docker_gwbridge
```

### Docker Swarm 

Create an overlay network:

```sh
docker network create \
  --driver overlay \
  netmon
```

Create dockerd-exporter global service (replace 172.18.0.1 with your docker_gwbridge address):

```sh
docker service create -d \
  --mode global \
  --name dockerd-exporter \
  --network netmon \
  -e IN="172.18.0.1:9323" \
  -e OUT="9323" \
  stefanprodan/dockerd-exporter:latest
```

Configure Prometheus to scrape the dockerd-exporter instances:

```
scrape_configs:
- job_name: 'dockerd-exporter'
  dns_sd_configs:
  - names:
    - 'tasks.dockerd-exporter'
    type: 'A'
    port: 9323
```

Run Prometheus on the same overlay network as dockerd-exporter.
