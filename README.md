Ansible Role: Couchpotato for Kubernetes
========================================

Ansible role to install Couchpotato on Kubernetes.

Role Variables
--------------

```yaml
# Image used
kubernetes_couchpotato_image: "linuxserver/couchpotato:latest"

# Namespace
kubernetes_couchpotato_namespace: "default"
# App name (used as selector)
kubernetes_couchpotato_app: "couchpotato"
# Deployment name
kubernetes_couchpotato_deployment: "couchpotato-deployment"
# Service name
kubernetes_couchpotato_service: "couchpotato"

# Number of replicas
kubernetes_couchpotato_replicas: 1
kubernetes_couchpotato_revision_history: 1

# Node selector
kubernetes_couchpotato_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_couchpotato_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_couchpotato_deployment_annotations: {}

kubernetes_couchpotato_resources:
  limits:
    memory: "1Gi"
  requests:
    memory: "256Mi"

# Setup ingress for couchpotato
kubernetes_couchpotato_setup_ingress: false
kubernetes_couchpotato_ingress:
  name: "couchpotato-ingress"
  host: "couchpotato.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /downloads/ (see examples for more details)
kubernetes_couchpotato_downloads_volumes: {}
# Movies volumes. Mounted in /movies/ (see examples for more details)
kubernetes_couchpotato_movies_volumes: {}
# Watch directories. Useful when blackhole is used. Mounted in /watchdirs/ (see
# examples for more details)
kubernetes_couchpotato_watchdirs_volumes: {}

# Couchpotato config volume. Contains the database, arts cache and config.
kubernetes_couchpotato_config_volume:
  definition:
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_couchpotato_downloads_volumes:
      # This volume will be mounted as /downloads/completed
      completed:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-torrents
            readOnly: false
        subPath: "completed"

    kubernetes_couchpotato_movies_volumes:
      # This volume will be mounted as /movies/global
      global:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-movies
            readOnly: false

    # Directories watched by our torrents client
    kubernetes_couchpotato_watchdirs_volumes:
      # This volume will be mounted as /watchdirs/providers
      providers:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-watchdirs
            readOnly: false
        subPath: "example/providers"

    kubernetes_couchpotato_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-couchpotato
          readOnly: false
        subPath: "config"

    kubernetes_couchpotato_setup_ingress: true
    kubernetes_couchpotato_ingress:
      name: "couchpotato-ingress"
      host: "couchpotato.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "couchpotato-ingress-tls"
          hosts:
            - "couchpotato.example.com"
  roles:
    - role: Anthony25.kubernetes-couchpotato
```

Use `run_once` to run the role on only one available master in the cluster.
If the couchpotato web interface is not reachable, please check that it is
listening on `0.0.0.0:5050`, and not only `localhost:5050` as the default is.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
