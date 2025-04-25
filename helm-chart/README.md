# ecr-proxy Helm Chart

[![Artifact HUB](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/evryfs-oss)](https://artifacthub.io/packages/search?repo=evryfs-oss)

Helm chart to deploy the ECR Proxy with local caching and access control to specific repositories. This proxy allows pulling Docker images from AWS ECR with caching capabilities.

---

## Installation

package it locally:

```sh
helm dependency update
helm package .
```

Then install the chart:

```sh
helm install ecr-proxy ./ecr-proxy -n ecr-proxy --create-namespace
```

Or install from the public ECR Helm registry:

```sh
helm install ecr-proxy oci://public.ecr.aws/i3y4r4k7/chart-ecr-proxy --version 1.0.13
```

---

## Values

Here is a sample `values.yaml` configuration:

```yaml
replicaCount: 2

image:
  repository: public.ecr.aws/i3y4r4k7/ecr-proxy
  tag: 1.0.13
  pullPolicy: IfNotPresent

serviceAccount:
  create: false  

env:
  - name: AWS_REGION
    value: "us-east-2"
  - name: AWS_USE_EC2_ROLE_FOR_AUTH
    value: "true"
  - name: UPSTREAM
    value: "https://825515450958.dkr.ecr.us-east-2.amazonaws.com"
  - name: RESOLVER
    value: "8.8.8.8"
  - name: AWS_ACCESS_KEY_ID
    valueFrom:
      secretKeyRef:
        name: ecr-proxy-aws-creds
        key: AWS_ACCESS_KEY_ID
  - name: AWS_SECRET_ACCESS_KEY
    valueFrom:
      secretKeyRef:
        name: ecr-proxy-aws-creds
        key: AWS_SECRET_ACCESS_KEY

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "1000m"
    memory: "1024Mi"

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /healthz
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: "30"  
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: "5"  
    alb.ingress.kubernetes.io/healthy-threshold: "3"  
    alb.ingress.kubernetes.io/unhealthy-threshold: "3"
    alb.ingress.kubernetes.io/acm-cert-arn: arn:aws:acm:us-east-2:825515450958:certificate/4bbc79a9-e2d7-4a4a-a746-63ced34fc8e9
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'  
    external-dns.alpha.kubernetes.io/hostname: docker-test.zeroq.cl
  hosts:
    - host: docker-test.zeroq.cl
      paths:
        - "/"

config:
  allowedRepos:
    - modulo-activity
    - bluequeen-api
    - bluequeen-colector
```

---

## Features

- 🔒 Restrict access to specific ECR repositories with `allowedRepos`.
- 💾 Local caching of Docker images using Nginx/OpenResty.
- 🌐 Ingress support with AWS ALB and TLS.
- ☁️ Works with EC2 IAM Roles or AWS credentials.
- 📦 Hosted publicly at [public.ecr.aws/i3y4r4k7/chart-ecr-proxy](https://public.ecr.aws/i3y4r4k7/chart-ecr-proxy)

---

## Maintainers

This chart is maintained by **Craftech**.

---

## License

MIT

