# Traefik

[Traefik](https://traefik.io/) is a modern HTTP reverse proxy and load balancer made to deploy
microservices with ease.

## User Notes
I have added a loadbalancer-values.yaml file, which contains the loadbalancer values settings from the Traefik chart, configuration equivalent to using --set service.type=LoadBalancer. To apply use the following install command
```bash
helm install traefik . --namespace traefik -f loadbalancer-values.yaml
```

## Custom Manifests

### Ingress rule for traefik to route to kong API Gateway

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kong-proxy-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: your-api-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kong-proxy  # This should match your Kong proxy service name
            port: 
              number: 8000  # This should match your Kong proxy http.servicePort
```

#### If we want an ingress with Cert Manager
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kong-proxy-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
    cert-manager.io/cluster-issuer: letsencrypt-prod
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: kong-proxy-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kong-proxy
            port:
              number: 8000
```



### Traefik Ingress with cert-manager
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: traefik
```


## Introduction

Starting with v28.x, this chart now bootstraps Traefik Proxy version 3 as a Kubernetes ingress controller,
using Custom Resources `IngressRoute`: <https://doc.traefik.io/traefik/v3.0/routing/providers/kubernetes-crd/>.

It's possible to use this chart with Traefik Proxy v2 using v27.x
This chart support policy is aligned with [upstream support policy](https://doc.traefik.io/traefik/deprecation/releases/) of Traefik Proxy.

See [Migration guide from v2 to v3](https://doc.traefik.io/traefik/v3.0/migration/v2-to-v3/) and upgrading section of this chart on CRDs.

### Philosophy

The Traefik HelmChart is focused on Traefik deployment configuration.

To keep this HelmChart as generic as possible we tend
to avoid integrating any third party solutions nor any specific use cases.

Accordingly, the encouraged approach to fulfill your needs:

1. Override the default Traefik configuration values ([yaml file or cli](https://helm.sh/docs/chart_template_guide/values_files/))
2. Append your own configurations (`kubectl apply -f myconf.yaml`)

[Examples](https://github.com/traefik/traefik-helm-chart/blob/master/EXAMPLES.md) of common usage are provided.

If needed, one may use [extraObjects](./traefik/tests/values/extra.yaml) or extend this HelmChart [as a Subchart](https://helm.sh/docs/chart_template_guide/subcharts_and_globals/).

## Installing

### Prerequisites

1. [x] Helm **v3 > 3.9.0** [installed](https://helm.sh/docs/using_helm/#installing-helm): `helm version`
2. [x] Traefik's chart repository: `helm repo add traefik https://traefik.github.io/charts`

### Kubernetes Version Support

Due to changes in CRD version support, the following versions of the chart are usable and supported on the following Kubernetes versions:

|                         |  Kubernetes v1.15 and below | Kubernetes v1.16-v1.21 | Kubernetes v1.22 and above |
|-------------------------|-----------------------------|------------------------|----------------------------|
| Chart v9.20.2 and below | [x]                         | [x]                    |                            |
| Chart v10.0.0 and above |                             | [x]                    | [x]                        |
| Chart v22.0.0 and above |                             |                        | [x]                        |

### CRDs Support of Traefik Proxy

Due to changes in API Group of Traefik CRDs from `containo.us` to `traefik.io`, this Chart install CRDs needed by default Traefik Proxy version, following this table:

|                         |  `containo.us`              | `traefik.io`           |
|-------------------------|-----------------------------|------------------------|
| Chart v22.0.0 and below |  [x]                        |                        |
| Chart v23.0.0 and above |  [x]                        | [x]                    |
| Chart v28.0.0 and above |                             | [x]                    |

### Deploying Traefik

```bash
helm install traefik traefik/traefik
```

or:

```bash
helm install traefik oci://ghcr.io/traefik/helm/traefik
```

You can customize the install with a `values` file. There are some [EXAMPLES](./EXAMPLES.md) provided.
Complete documentation on all available parameters is in the [default file](./traefik/values.yaml).

```bash
helm install -f myvalues.yaml traefik traefik/traefik
```

🛂 **Warning**: Helm v2 support was removed in the chart version 10.0.0.

## Upgrading

One can check what has changed in the [Changelog](./traefik/Changelog.md).

:information_source: With Helm v3, CRDs created by this chart can not be updated, cf. the [Helm Documentation on CRDs](https://helm.sh/docs/chart_best_practices/custom_resource_definitions).

:warning: Please read carefully release notes of this chart before upgrading CRDs.

```bash
# Update repository
helm repo update
# See current Chart & Traefik version
helm search repo traefik/traefik
# Update CRDs (Traefik Proxy v3 CRDs)
kubectl apply --server-side --force-conflicts -k https://github.com/traefik/traefik-helm-chart/traefik/crds/
# Upgrade Traefik
helm upgrade traefik traefik/traefik
```

New major version indicates that there is an incompatible breaking change.

#### Upgrade up to 27.X

When upgrading on Traefik Proxy v2 version, one need to stay at Traefik Helm Chart v27.x. The command to upgrade to the latest Traefik Proxy v2 CRD is:

```bash
kubectl apply --server-side --force-conflicts -k https://github.com/traefik/traefik-helm-chart/traefik/crds/?ref=v27
```

### Upgrading after 18.X+

It's detailed in [release notes](https://github.com/traefik/traefik-helm-chart/releases).

### Upgrading from 17.x to 18.x

Since v18.x, this chart by default merges TCP and UDP ports into a single (LoadBalancer) `Service`.
Load balancers with mixed protocols are available since v1.20 and in
[beta as of Kubernetes v1.24](https://kubernetes.io/docs/concepts/services-networking/service/#load-balancers-with-mixed-protocol-types).
Availability may depend on your Kubernetes provider.

To retain the old default behavior, set `service.single` to `false` in your values.

When using TCP and UDP with a single service, you may encounter
[this issue](https://github.com/kubernetes/kubernetes/issues/47249#issuecomment-587960741)
from Kubernetes.

On HTTP/3, if you want to avoid this issue, you can set
`ports.websecure.http3.advertisedPort` to an other value than `443`

If you were previously using HTTP/3, you should update your values as follows:
  - Replace the old value (`true`) of `ports.websecure.http3` with a key `enabled: true`
  - Remove `experimental.http3.enabled=true` entry

### Upgrading from 16.x to 17.x

Since v17.x, this chart provides unified labels following
[Kubernetes recommendation](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/).

This version needs to change an immutable field, which is not supported by
Kubernetes and Helm, see [this issue](https://github.com/helm/helm/issues/7350)
for more details.
So you will have to delete your `Service`,  `Deployment` or `DaemonSet` in
order to be able to upgrade.

You may also upgrade by deploying another Traefik to a different namespace and
removing after your first Traefik.

Alternatively, since version 20.3.0 of this chart, you may set `instanceLabelOverride` to the previous value of that label.
This will override the new `Release.Name-Release.Namespace` pattern to avoid any (longer) downtime.

## Contributing

If you want to contribute to this chart, please read the [Contributing Guide](./CONTRIBUTING.md).

Thanks to all the people who have already contributed!

<a href="https://github.com/traefik/traefik-helm-chart/graphs/contributors">
  <img src="https://contributors-img.web.app/image?repo=traefik/traefik-helm-chart" />
</a>
