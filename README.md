# Deployment for ingestd in the Rubin US Data Facility

This repository contains Kubernetes deployment YAML files and configuration YAML files for the Rucio-to-Data Butler ingest daemon that is part of the Rubin US Data Facility hosted at SLAC.

# SLAC National Accelerator Laboratory
The SLAC National Accelerator Laboratory is operated by Stanford University for the US Department of Energy.  

# Background

Rucio, via the Hermes-K daemon, sends messages on Kafka topics specific to each configured Rucio Storage Element (RSE) when replicas have been created.
The ingest daemon (lsst-dm/ctrl\_ingestd) listens for these messages and ingests the corresponding file into the configured LSST Science Pipelines Data Butler, using metadata in the message.

# Deployment Structure

ingestd is deployed as a Kubernetes Deployment, which spawns a ReplicaSet.
Multiple ingestds can run on the same Kafka topic in parallel, as they share a consumer group.

# Deployment Process

Log into the k8s vcluster (usdf-ingestd).

Obtain a token to access the secrets in vault:

```
# obtain token to access secrets
export VAULT_ADDR=https://vault.slac.stanford.edu
vault login -method ldap -username <username>
```

Apply the Kubernetes manifests:

```
cd kubernetes/overlays/<environment>
make apply
```

The above will authenticate you against our vault instance so that you can obtain the most up-to-date secrets, download the passwords temporarily into your working directory, push the Kubernetes manifests to the cluster and then subsequently remove the secrets.
