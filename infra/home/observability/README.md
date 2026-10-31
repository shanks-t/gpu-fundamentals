# Retired — home observability is owned by the GPU Server Home Lab repository

This curriculum copy of the shared observability stack was retired in the
D037 consolidation. Its Compose file, configs, dashboards, and provisioning
are **no longer maintained here and must not be started again** — the files
were removed from this repository to prevent an accidental duplicate stack.

The single shared stack for the whole lab (Prometheus, Grafana on tailnet
port 3000, node_exporter, DCGM exporter, plus the Alloy collector and Tempo
trace backend) is owned and run from:

- Repository: <https://github.com/shanks-t/gpu-server-home-lab> (private;
  server checkout `/home/trey/workspace/gpu-server-home-lab`)
- Compose project: `home-observability` — `infra/observability/compose.yaml`

Ownership rules, resource budgets, and the permanent Grafana tailnet HTTPS
URL (`https://gpu-server.tail92f21a.ts.net/` — the Tailscale proxy for the
host's loopback port 3000) live in that repository's
`infra/observability/README.md`. Shared monitoring is changed
only there, via that repository's own reviewed Git workflow — never from
this curriculum checkout.
