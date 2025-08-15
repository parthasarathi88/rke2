# Longhorn Storage Provisioner Role

This Ansible role installs and configures Longhorn as a distributed block storage solution for Kubernetes clusters.

## Features
- Installs Longhorn via Helm
- Sets Longhorn as the default storage class
- Waits for Longhorn pods to be ready

## Variables
- `longhorn_chart_version`: Helm chart version (default: 1.6.2)

## Usage
Add `longhorn` to your playbook roles:
```yaml
roles:
  - longhorn
```

## References
- [Longhorn Documentation](https://longhorn.io/docs/)
- [Longhorn Helm Chart](https://artifacthub.io/packages/helm/longhorn/longhorn)
