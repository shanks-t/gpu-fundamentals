# Home GPU observability

This independent Compose project monitors the home GPU server with DCGM
Exporter, node_exporter, Prometheus, and Grafana. All backend HTTP endpoints
bind only to remote loopback. Tailscale Serve proxies Grafana over private
tailnet HTTPS; never use Tailscale Funnel or open router ports.

## Start

Create the Grafana administrator secret on the server without printing it:

```sh
install -d -m 0700 "$HOME/.config/gpu-observability"
umask 077
openssl rand -base64 32 >"$HOME/.config/gpu-observability/grafana_admin_password"
docker compose -f infra/home/observability/compose.yaml up -d
```

## Access from the tailnet

Tailscale Serve is configured persistently on `gpu-server`:

```sh
sudo tailscale serve --bg 3000
tailscale serve status
```

From any device connected to the same tailnet, open:

```text
https://gpu-server.tail92f21a.ts.net/
```

Sign in as `admin`, and read the generated password in a separate local
terminal only when needed:

```sh
ssh -t gpu-home 'cat "$HOME/.config/gpu-observability/grafana_admin_password"'
```

Anonymous access and user sign-up are disabled. Tailscale controls network
reachability; Grafana authentication remains required. Disable the proxy with
`sudo tailscale serve --https=443 off`.

If Serve is unavailable, use an SSH tunnel as a fallback:

```sh
ssh -N -L 3000:127.0.0.1:3000 gpu-home
```

Then open `http://127.0.0.1:3000`.

The provisioned **GPU Server Overview** dashboard includes GPU utilization,
temperature, power, framebuffer memory, CPU, RAM, storage, and wired Ethernet
throughput.

## Storage monitoring and host limits

The root-filesystem panel uses green below 70%, yellow from 70% to 80%, red
from 80% to 90%, and dark red at 90% and above. Prometheus evaluates matching
alerts after 30, 15, and 5 minutes respectively, plus a critical alert when
less than 20 GiB remains. Alerts are visible in Prometheus; outbound
notifications require a separately configured Grafana or Alertmanager contact
point so credentials remain outside this repository.

Prometheus retains at most 30 days or 5 GB, whichever limit is reached first.
Install the version-controlled Docker and journald limits on the host with:

```sh
sudo install -m 0644 infra/home/host-config/docker-daemon.json /etc/docker/daemon.json
sudo install -d -m 0755 /etc/systemd/journald.conf.d
sudo install -m 0644 infra/home/host-config/journald-storage.conf \
  /etc/systemd/journald.conf.d/60-gpu-home-storage.conf
sudo systemctl restart systemd-journald
sudo systemctl restart docker
```

Docker retains at most three 50 MB JSON log files per newly created container;
recreate existing containers to adopt that default. Journald is capped at 1 GB
of persistent storage and 256 MB of runtime storage. Do not automate Docker
image or build-cache deletion: inspect `docker system df` and remove only
reproducible, disposable data after an alert.

When the data SSD is later mounted, set `GPU_LAB_CACHE_ROOT` beneath
`/srv/gpu-lab`. The home Compose wrapper refuses to create cache directories
there unless `/srv/gpu-lab` is an actual mount, preventing a missing data drive
from silently filling the root filesystem.

## Pinned images

- DCGM Exporter `4.5.3-4.8.2-distroless` — `sha256:60d3b00ac80b4ae77f94dae2f943685605585ad9e92fdccda3154d009ae317cc`
- Prometheus `3.13.0 LTS` — `sha256:c6b27ea434f8389bfe233fbc7be381cf50587c286e871bc842008f5a1b1908a7`
- node_exporter `1.11.1` — `sha256:e9cff4fc67b1818f8c97adb115b9f12c9a54b533de86765d4a0effc01b357205`
- Grafana `13.2.1` — `sha256:f772d434e8fab0049deb2b1b30abd43342bcfca1537614aa8d36080232cf4283`
