# Deployment for transformed EFD in the Rubin US Data Facility

This repository contains Kubernetes manifests for transformed EFD processing at the Rubin US Data Facility.

# Background

The processor reads EFD data and writes transformed records to ConsDB. It uses a Butler repository for instrument data.

# Deployment Structure

The `kubernetes/base` directory defines shared Job and CronJob templates.

The `kubernetes/overlays/dev` and `kubernetes/overlays/prod` directories define instrument-specific deployments for LATISS, LSSTCam, and LSSTComCam.

Each overlay uses Kustomize to generate a Kubernetes secret from Vault values. The generated secret is removed after `make dump` or `make apply`.

# Deployment Process

Log into the target Kubernetes vcluster.

Obtain a Vault token:

```
export VAULT_ADDR=https://vault.slac.stanford.edu
vault login -method ldap -username <username>
```

Render or apply one overlay:

```
cd kubernetes/overlays/<environment>/<instrument>-<job-or-cronjob>
make dump file=<manifest>.yaml embargo=true
make apply file=<manifest>.yaml embargo=true
```

Set `embargo=false` to use the main Butler account. Use `job.yaml`, `cronjob.yaml`, or `failure-monitor.yaml` when the file exists in the overlay.

Do not commit files under `etc/.secrets/`.
