# Ingress & Traffic Routing in Kubernetes

This guide covers how to expose HTTP(S) applications running in Kubernetes to the outside world using Ingress — the layer that sits in front of your Services and routes traffic by hostname and path.

## Why Ingress?

Each Kubernetes Service exposure type has trade-offs:

| Method | What you get | Drawback |
|--------|--------------|----------|
| `ClusterIP` | Internal virtual IP | Not reachable from outside |
| `NodePort` | Static port (30000–32767) on every node | Ugly ports, no host/path routing, no TLS termination |
| `LoadBalancer` | A real cloud load balancer with an external IP | One cloud LB (≈ cost) **per Service** |
| **`Ingress`** | **One entry point routing many Services by host/path, with TLS** | Needs an *Ingress Controller* installed |

An **Ingress** object by itself does nothing — it is just a routing rule set. An **Ingress Controller** (ingress-nginx, Traefik, HAProxy, Cilium, ...) watches Ingress objects and programs an actual load balancer to implement them.

## Prerequisites

- A running cluster (Minikube is perfect for this)
- An Ingress Controller:

```bash
# Minikube — one command
minikube addons enable ingress

# Any cluster with Helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx
```

## Files in this folder

| File | Purpose |
|------|---------|
| `ingress-demo.yaml` | Two tiny apps (`coffee`, `tea`) + Services + one Ingress routing by path |

## Walkthrough

### 1. Deploy the demo

```bash
kubectl apply -f ingress-demo.yaml
kubectl get ingress
# NAME           CLASS   HOSTS        ADDRESS        PORTS
# cafe-ingress   nginx   cafe.local   192.168.49.2   80
```

Wait until `ADDRESS` shows an IP (the ingress controller's endpoint).

### 2. Route the hostname to that address

Add an entry to your `/etc/hosts` (Windows: `C:\Windows\System32\drivers\etc\hosts`):

```bash
echo "$(minikube ip) cafe.local" | sudo tee -a /etc/hosts
```

Or, without touching `/etc/hosts`, use `curl --resolve`:

```bash
curl --resolve cafe.local:80:$(minikube ip) http://cafe.local/coffee
# coffee-v1
curl --resolve cafe.local:80:$(minikube ip) http://cafe.local/tea
# tea-v1
```

### 3. What just happened?

1. `curl` resolved `cafe.local` to the ingress controller's IP
2. The controller matched `Host: cafe.local` against the Ingress rule
3. `/coffee` prefix → `coffee-svc` → pods labelled `app: coffee`
4. `/tea` prefix → `tea-svc` → pods labelled `app: tea`

## Key concepts

### `pathType`

| Value | Meaning |
|-------|---------|
| `Prefix` | Matches the path and everything under it (`/tea` matches `/tea/hot`) |
| `Exact` | Matches only the exact path |
| `ImplementationSpecific` | The controller decides (e.g. regex support in nginx) |

### `ingressClassName`

Modern clusters route Ingress objects to controllers via `spec.ingressClassName` (matches the controller's ingress class, usually `nginx`). Older setups used the deprecated `kubernetes.io/ingress.class` annotation — avoid it.

### Multiple hosts

One Ingress can carry many host rules, and several Ingresses can share one controller — this is how a single external IP serves a whole team's apps:

```yaml
rules:
- host: app1.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: app1-svc
          port:
            number: 80
- host: app2.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: app2-svc
          port:
            number: 80
```

## TLS termination

Add a Secret containing a certificate and a `tls` section — the controller terminates TLS for you:

```bash
# quick self-signed cert for local testing
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=cafe.local"

kubectl create secret tls cafe-tls --cert=tls.crt --key=tls.key
```

```yaml
spec:
  tls:
  - hosts:
    - cafe.local
    secretName: cafe-tls
  rules:
  - host: cafe.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: coffee-svc
            port:
              number: 80
```

> 💡 For real certificates, install [cert-manager](https://cert-manager.io/) — it issues and renews Let's Encrypt certs automatically.

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| Ingress has no `ADDRESS` | No controller installed, or `ingressClassName` mismatch |
| Connection refused | Traffic not reaching the controller (firewall/hosts entry) |
| 404 from the controller | Host header doesn't match any `rules[].host`, or path typo |
| Default backend / 503 | Service exists but no pods match its selector (check `kubectl get endpoints`) |

```bash
kubectl get ingressclasses                # which controllers are available?
kubectl -n ingress-nginx get pods         # controller healthy?
kubectl describe ingress cafe-ingress     # events at the bottom show routing errors
kubectl get endpoints coffee-svc          # empty = selector doesn't match any pod
```

## Additional Resources

- [Ingress Concepts](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [ingress-nginx Documentation](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager](https://cert-manager.io/docs/)
- [Gateway API](https://gateway-api.sigs.k8s.io/) — the successor to Ingress, worth knowing about
