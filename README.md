# open-balena-helm
OpenBalena Kubernetes Helm repository

## Component selection

You can enable or disable components via `values.yaml` or `--set`:

| Flag | Components |
|------|------------|
| `api.enabled` | open-balena-api |
| `registry.enabled` | open-balena-registry |
| `vpn.enabled` | open-balena-vpn |
| `ui.enabled` | UI, PostgREST, and Remote (one flag for all three) |

Example: API and registry only (no VPN, no UI):

```yaml
api:
  enabled: true
registry:
  enabled: true
vpn:
  enabled: false
ui:
  enabled: false
```

When `ui.enabled` is false, set `haproxy.controller.tcp` to `{}` in your values (or override) so HAProxy does not reference the remote service. When `vpn.enabled` is false, override `haproxy.controller.config.config-proxy` to remove the VPN redirect block.

### UI-only with AWS ALB (no HAProxy, no chart certs)

For UI-only you can use the AWS Load Balancer Controller and external-dns instead of HAProxy and the chart’s PKI certs:

- Set `haproxy.enabled: false` so the HAProxy subchart is not installed.
- Set `global.hostname` to your domain (e.g. `openbalena.example.com`).
- Set `ingress.ingressClassName` to your ALB ingress class (e.g. `alb`).
- Add `ingress.annotations` for external-dns and any ALB annotations (e.g. `alb.ingress.kubernetes.io/scheme: internet-facing`, and external-dns hostname annotations for your domain).
- No PKI secrets are created when API, registry, and VPN are disabled; use cert-manager with your ingress class and `issuers.acme.enabled: true`, or terminate TLS on the ALB (e.g. AWS ACM).

The chart still creates one HTTP Ingress for admin, postgrest, and remote when `ui.enabled` is true and `global.hostname` is set; your ingress controller (e.g. ALB) will serve it.

Requirements:

Docker w/ kubernetes enabled
Install docker
Settings -> Kubernetes -> Enable Kubernetes

kubectl
https://kubernetes.io/docs/tasks/tools/

helm
https://helm.sh/docs/intro/install/


Installation:

git clone https://github.com/dcaputo-harmoni/open-balena-helm.git
git submodule update --init --recursive

Forward *.openbalena.yourdomain.com to the public IP address of your docker host

Run scripts/install-cert-manager.sh
Run scripts/install-openbalena.sh regenerate-config <admin password> <db password>
