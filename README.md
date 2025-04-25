<p align="left">
    <a href="https://hub.docker.com/r/craftech/aws-ecr-http-proxy" alt="Pulls">
        <img src="https://img.shields.io/docker/pulls/craftech/aws-ecr-http-proxy" /></a>
    <a href="https://www.craftech.io" alt="Maintained">
        <img src="https://img.shields.io/maintenance/yes/2025.svg" /></a>
</p>

# aws-ecr-http-proxy

A simple, production-ready Nginx proxy that caches and forwards requests to **AWS ECR**, including support for **public and private ECR**. Enhanced by [Craftech](https://www.craftech.io), the proxy includes fine-grained access control, optional authentication, and Helm deployment support.

---

### New Features:

- ✅ **Fine-grained access control**: define a list of allowed repositories per proxy instance via `ALLOWED_REPOSITORIES`.
- ✅ **Public ECR support**: works with `public.ecr.aws`.
- ✅ **Optimized for Docker clients**: fully compatible with Docker pull/push workflows.
- ✅ **Flexible deployment**: includes Docker, Kubernetes (Helm), and Ansible options.
- ✅ **Improved logging and metrics support** (WIP).
- ✅ **Modular Lua-based proxy logic** using OpenResty.
- ✅ **Maintained by [Craftech](https://www.craftech.io)** as of 2025.

---

### Configuration

The proxy is packaged as a Docker container and can be configured using the following environment variables:

| Environment Variable                | Description                                                                   | Status                            | Default    |
| :---------------------------------:| :-----------------------------------------------------------------------------| :-------------------------------: | :--------: |
| `AWS_REGION`                       | AWS Region for AWS ECR                                                        | Required                          |            |
| `AWS_ACCESS_KEY_ID`               | AWS Account Access Key ID                                                     | Optional                          |            |
| `AWS_SECRET_ACCESS_KEY`           | AWS Account Secret Access Key                                                 | Optional                          |            |
| `AWS_USE_EC2_ROLE_FOR_AUTH`       | Use EC2 IAM Role for authentication                                           | Optional                          | `false`    |
| `UPSTREAM`                         | Base URL for AWS ECR (private or public)                                      | Required                          |            |
| `RESOLVER`                         | DNS resolver for Nginx                                                        | Required                          |            |
| `PORT`                             | Proxy listening port                                                           | Required                          |            |
| `CACHE_MAX_SIZE`                   | Maximum cache volume size                                                     | Optional                          | `75g`      |
| `CACHE_KEY`                        | Cache key for content                                                         | Optional                          | `$uri`     |
| `ENABLE_SSL`                       | Enable SSL/TLS                                                                | Optional                          | `false`    |
| `REGISTRY_HTTP_TLS_KEY`           | TLS key path inside container                                                 | Required with TLS                 |            |
| `REGISTRY_HTTP_TLS_CERTIFICATE`   | TLS cert path inside container                                                | Required with TLS                 |            |
| `ALLOWED_REPOSITORIES`            | Comma-separated list of repos to allow (e.g. `nginx,redis,zeroq/*`)           | Optional                          | (Allow all)|

---

### Example (Docker)

```sh
docker run -d --name docker-registry-proxy --net=host \
  -v /registry/local-storage/cache:/cache \
  -v /registry/certificate.pem:/opt/ssl/certificate.pem \
  -v /registry/key.pem:/opt/ssl/key.pem \
  -e PORT=5000 \
  -e RESOLVER=8.8.8.8 \
  -e UPSTREAM=https://public.ecr.aws/a5w2e8z7 \
  -e AWS_REGION=us-east-1 \
  -e ENABLE_SSL=true \
  -e REGISTRY_HTTP_TLS_KEY=/opt/ssl/key.pem \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/opt/ssl/certificate.pem \
  -e ALLOWED_REPOSITORIES=zeroq/*,nginx,alpine \
  craftech/aws-ecr-http-proxy:latest
```

You can now pull from your proxy like:

```sh
docker pull docker.zeroq.cl:5000/zeroq/admin:latest
```

---

### Deploying the proxy

#### On Kubernetes (Helm)

> A Helm chart is available and maintained for this version.

```bash
helm repo add craftech https://charts.craftech.io
helm install ecr-proxy craftech/ecr-proxy -n ecr-proxy --create-namespace \
  -f values.yaml
```

Refer to the chart documentation for a full list of configurable values.

---

### SSL/TLS Options

To enable encrypted HTTPS access to your proxy:

1. Set `ENABLE_SSL=true`
2. Mount your valid `.pem` files into the container
3. Provide the correct environment variables:
   - `REGISTRY_HTTP_TLS_KEY`
   - `REGISTRY_HTTP_TLS_CERTIFICATE`

Alternatively, for testing purposes, mark the proxy host as **insecure** in your Docker client configuration:

```json
{
  "insecure-registries": ["docker.zeroq.cl:5000"]
}
```

---

### Maintainers

Originally developed by [eSailors](https://www.esailors.de), now maintained and extended by [Craftech](https://www.craftech.io).

For contributions, issues, or feature requests, please open a ticket or PR on GitHub.

