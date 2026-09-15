# Home GPU observability

This independent Compose project monitors the home GPU server with DCGM
Exporter, node_exporter, Prometheus, and Grafana. All HTTP endpoints bind only
to remote loopback. Access Grafana through the `gpu-home` SSH connection; never
open router ports.

## Start

Create the Grafana administrator secret on the server without printing it:

```sh
install -d -m 0700 "$HOME/.config/gpu-observability"
umask 077
openssl rand -base64 32 >"$HOME/.config/gpu-observability/grafana_admin_password"
docker compose -f infra/home/observability/compose.yaml up -d
```

Forward Grafana from the Mac:

```sh
ssh -N -L 3000:127.0.0.1:3000 gpu-home
```

Open `http://127.0.0.1:3000`, sign in as `admin`, and read the generated
password in a separate local terminal only when needed:

```sh
ssh -t gpu-home 'cat "$HOME/.config/gpu-observability/grafana_admin_password"'
```

Anonymous access and user sign-up are disabled. The provisioned **GPU Server
Overview** dashboard includes GPU utilization, temperature, power, framebuffer
memory, CPU, RAM, storage, and wired Ethernet throughput.

## Pinned images

- DCGM Exporter `4.5.3-4.8.2-distroless` — `sha256:60d3b00ac80b4ae77f94dae2f943685605585ad9e92fdccda3154d009ae317cc`
- Prometheus `3.13.0 LTS` — `sha256:c6b27ea434f8389bfe233fbc7be381cf50587c286e871bc842008f5a1b1908a7`
- node_exporter `1.11.1` — `sha256:e9cff4fc67b1818f8c97adb115b9f12c9a54b533de86765d4a0effc01b357205`
- Grafana `13.2.1` — `sha256:f772d434e8fab0049deb2b1b30abd43342bcfca1537614aa8d36080232cf4283`
