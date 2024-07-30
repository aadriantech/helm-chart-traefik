# Changelog

> Date: 20240730
> values-metallb-traefik-kong.yaml

🚨 When enabling persistence for certificates, permissions on acme.json can be
lost when Traefik restarts. You can ensure correct permissions with an
initContainer. See https://github.com/traefik/traefik-helm-chart/blob/master/EXAMPLES.md#use-traefik-native-lets-encrypt-integration-without-cert-manager
for more info. 🚨

### TLS Options and Store

These sections are kept as default, as they are essential for setting up TLS options and stores but do not require specific modifications for your architecture.

### Service Configuration

- Enabled the Traefik service.
- Configured Traefik to use a single service for both TCP and UDP to align with your architecture.
- Set Traefik to use a LoadBalancer, which is essential for integration with MetalLB.
- Added placeholders for annotations to both TCP and UDP services for future use.
- Kept annotationsTCP and annotationsUDP empty for customization if needed.
- Kept labels empty for adding custom labels if needed.
- Kept spec empty for additional entries if needed.
- Left externalTrafficPolicy, loadBalancerIP, and clusterIP as comments for potential configuration.
- Kept loadBalancerSourceRanges empty for security settings.
- Kept externalIPs empty for specifying external IPs if needed.
- Kept additionalServices empty for adding additional internal services if required.

### Autoscaling

Disabled autoscaling as it is not specified in your architecture requirements.

### Persistence

- Disabled persistence as it is not required for Traefik in your architecture.
- Kept other configurations as default with `name: data`, `accessMode: ReadWriteOnce`, `size: 128Mi`, and default path `/data`.

### Cert Resolvers

Kept default configuration for certificate resolvers, no specific modifications required.

### Host Network

Ensured Traefik does not use the host network namespace, which is suitable for your Kubernetes setup.

### RBAC

- Ensured RBAC is enabled.
- Configured to use ClusterRole and ClusterRoleBinding for cross-namespace access.
- Kept secretResourceNames empty for specifying Kubernetes secrets accessible to Traefik.

### Pod Security Policy

Disabled PodSecurityPolicy as it is not explicitly required.

### Service Account

Left service account name empty to allow automatic creation using the fullname template.

### Service Account Annotations

Kept default with no additional annotations.

### Resources

Kept default with no specific resource requests or limits.

### Affinity

Kept empty but provided an example for pod anti-affinity.

### Node Selector

Kept default with no specific node selection constraints.

### Tolerations

Kept default with no specific tolerations.

### Topology Spread Constraints

Kept empty but provided an example for controlling pod distribution.

### Priority Class Name

Kept default with no specific priority class.

### Security Context

Configured secure container settings: `allowPrivilegeEscalation: false`, `capabilities: drop: [ALL]`, `readOnlyRootFilesystem: true`.

### Pod Security Context

Configured with secure defaults: `runAsGroup: 65532`, `runAsNonRoot: true`, `runAsUser: 65532`.

### Extra Objects

Kept default for deploying extra Kubernetes resources if needed.

### Namespace Override

Left empty for potential namespace override during Helm deployment.

### Instance Label Override

Left empty for overriding the default instance label if required.

### Traefik Hub Configuration

- Provided default settings for Traefik Hub.
- Set API Management to disabled.
- Configured Redis settings with default values, enabled Redis cluster.