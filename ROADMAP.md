# Roadmap

The cluster from [k8s-infra](https://github.com/edy-rus/k8s-infra) is up but empty. This repo fills it, in three stages: the foundation that lets Argo CD manage everything, then observability, then apps once there is more hardware.

The one idea to hold on to: **nothing is installed by hand.** After the one bootstrap step, every change is a commit here, and Argo CD makes the cluster match. A reset in k8s-infra reinstalls Argo CD, and Argo CD rebuilds everything else from this repo.

```mermaid
flowchart TB
    found["<b>Foundation</b><br/>Argo CD, storage,<br/>private access"]
    obs["<b>Observability</b><br/>metrics, logs,<br/>traces"]
    practice["<b>Practice project</b><br/>a service to run,<br/>watch and break"]
    apps["<b>Apps</b><br/>later, with<br/>more hardware"]

    found --> obs --> practice --> apps
```

## Constraints

These shape every choice below:

- **One small node.** The lab is a single Talos VM on chibang, which has 4 cores and 32 GB of RAM in total. Everything must fit next to the storage host's own work.
- **The network can change.** chibang may move between sites, so nothing should depend on a LAN address range. The cluster should work at any site with DHCP and a tailnet.
- **Storage stays on chibang.** The cluster only keeps data it can lose, such as metrics and logs with short retention. Apps that need real data wait.

## Foundation

- [x] **Bigger lab VM.** `talos-cp1` raised from 2 vCPU and 6 GB to 4 vCPU and 12 GB in k8s-infra, so the observability stack fits. The resize wiped the VM's Secure Boot keys, so the node was reset afterwards (see the k8s-infra README).
- [x] **Argo CD bootstrap** as the last OpenTofu step in k8s-infra: install Argo CD with Helm, using the values from this repo, and create the root application pointing at `clusters/lab/`. Argo CD now manages its own upgrades, and `reset.sh` reinstalls it.
- [x] **App of apps.** One root application, one child application per component, each with pinned chart versions and its values in Git. Working, with Argo CD as the first child.
- [ ] **Storage for throwaway data.** A local-path provisioner on the Talos VM disk, for Prometheus and Loki. Losing it means losing recent metrics and logs, nothing more.
- [ ] **Private access over Tailscale.** The Tailscale Kubernetes operator gives each UI (Argo CD, Grafana) its own tailnet name with HTTPS. It works the same from any site, with no LAN IPs, no load balancer and nothing exposed to the internet.
- [ ] **Secrets that survive a reset.** Encrypted secrets in Git (Sealed Secrets or SOPS with age), starting with the Tailscale OAuth client. The one decryption key lives in my password store, and k8s-infra puts it into the cluster during bootstrap, before Argo CD starts. Then `reset.sh` brings everything back with no manual steps, because Git holds everything else.

## Observability

- [ ] **Metrics:** kube-prometheus-stack (Prometheus, Alertmanager, Grafana, node and cluster dashboards). Short retention and modest resource limits.
- [ ] **Logs:** Loki in single-binary mode, with Grafana Alloy collecting pod logs.
- [ ] **Traces:** an OpenTelemetry Collector, plus Tempo to store traces, all viewed in Grafana.
- [ ] **Alerts that reach me:** Alertmanager to a phone notification for a short list of real problems (node down, disk filling, Argo CD out of sync).
- [ ] **Dashboards as code:** Grafana dashboards stored in this repo, not built by clicking.

## Then

- [ ] **The practice project** deploys here and uses the observability stack for its traces, metrics and load tests.
- [ ] **Write-up** for the portfolio once the foundation and observability work.

## Apps (Later)

Waiting on more hardware and a storage decision. The wish list, from earlier planning: Immich (first, to leave Google Photos), Jellyfin, Navidrome, Open WebUI, Gitea, Nextcloud, Pi-hole. Apps that need real data will read it from chibang, never keep the only copy inside the cluster.
