# Strimzi-Kafka GitOps Setup

This repository contains the GitOps configuration for deploying Strimzi-Kafka on a Kubernetes cluster using Argo CD.

## Overview

This setup automates the installation and configuration of:
- **Strimzi Kafka Operator** - Manages Kafka cluster lifecycle
- **Vault** - Secret management with KV v2 and PKI secrets engine
- **External Secrets Operator** - Syncs secrets from Vault to Kubernetes
- **Keycloak** - Identity and Access Management (IAM) for OAuth (pending)
- **Argo CD** - GitOps operator for continuous deployment

## Repository Structure

```
kafka-gitops/
├── argo-app.yaml              # Argo CD Application manifest
├── kafka.yaml                 # Kafka Cluster CR (Strimzi v1)
├── todo_strimzi.txt           # Task tracking file
└── manifests/
    ├── argo-app.yaml          # Argo CD Application manifest (duplicate)
    ├── external-secrets/
    │   ├── secretstore.yaml   # External Secrets SecretStore
    │   └── external-secrets/  # External Secrets definitions
    │       ├── kafka-tls.yaml
    │       └── keycloak-client-secret.yaml
    ├── kafka/
    │   └── kafka.yaml         # Kafka Cluster CR
    ├── keycloak/
    │   └── keycloak.yaml      # Keycloak deployment
    ├── strimzi/
    │   ├── strimzi-operator.yaml  # Strimzi operator manifests
    │   └── crds/                  # Strimzi CRDs
    └── vault/
        ├── vault.yaml           # Vault deployment (dev mode)
        └── vault-kv-pki.yaml    # Vault KV v2 and PKI configuration
```

## Prerequisites

- Kubernetes cluster (v1.28+ recommended)
- `kubectl` configured with cluster admin access
- `argocd` CLI installed
- Network access to Docker Hub for container images
- SSH key configured for GitHub access

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/funnyman7777-hash/kafka-gitops.git
cd kafka-gitops
```

### 2. Apply Argo CD Application

```bash
kubectl apply -f manifests/argo-app.yaml
```

### 3. Verify Argo CD Installation

```bash
kubectl get pods -n argocd
```

**Admin Password:** `3GNwJAJAZOGChLzn`

### 4. Verify All Components

```bash
# Strimzi Operator
kubectl get pods -n strimzi

# Kafka Cluster
kubectl get pods -n kafka

# Vault
kubectl get pods -n vault

# External Secrets
kubectl get pods -n external-secrets
```

## Secrets Management

### Vault Configuration

- **Vault Token:** `my-vault-token-123`
- **KV Path:** `secret/data/kafka/tls` (TLS secrets)
- **SecretStore:** `vault-secretstore` (ClusterSecretStore in kafka namespace)

### External Secrets

- **kafka-tls:** Syncs TLS secrets from Vault to Kafka namespace
- **keycloak-client-secret:** Syncs Keycloak client secrets (pending)

## Kafka Cluster Configuration

- **Kafka Version:** 4.3.1
- **Brokers:** 4
- **Controllers:** 3
- **Authentication:** Mutual TLS (strimzi-mtls)
- **Encryption:** TLS (strimzi-tls)

### Bootstrap Server

```
my-cluster-kafka-bootstrap.kafka.svc:9093  # TLS
my-cluster-kafka-bootstrap.kafka.svc:9092  # Plain
```

## Admin Passwords

| Service | Username | Password |
|---------|----------|----------|
| Argo CD | admin | `3GNwJAJAZOGChLzn` |
| Vault | N/A | `my-vault-token-123` |
| Keycloak | admin | `adminpassword` (pending) |

## Testing

### Install Kafka Client Tools

```bash
# Install Java 21
sudo apt-get install -y openjdk-21-jre-headless

# Download Kafka
wget https://archive.apache.org/dist/kafka/4.3.1/kafka_2.13-4.3.1.tgz
tar -xzf kafka_2.13-4.3.1.tgz
cd kafka_2.13-4.3.1/bin/
```

## Troubleshooting

### Argo CD Issues

```bash
kubectl get pods -n argocd
```

### Vault Issues

```bash
kubectl exec -n vault vault-0 -- vault status
```

### Kafka Issues

```bash
kubectl get pods -n kafka
kubectl describe kafkas.kafka.strimzi.io my-cluster -n kafka
```

## Notes

- **Keycloak:** Keycloak deployment is pending due to network connectivity issues preventing image pulls from Docker Hub.
- **OAuth:** OAuth configuration requires Keycloak to be running first. Current setup uses mutual TLS for authentication.
- **TLS:** All communication is encrypted using TLS. Admin certificates are required for Kafka client authentication.

## GitOps Workflow

1. Changes are made to the GitHub repository
2. Argo CD detects the changes and syncs them to the cluster
3. All Kubernetes manifests are managed through GitOps

## License

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.

## Contact

For questions or support, please open an issue in the GitHub repository.

---

*Last updated: 2026-10-23*
