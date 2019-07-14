# Kustomization fails with version >= 3.0.0 because namespace

## Pointers

- see <https://github.com/kubernetes-sigs/kustomize/issues/1332>

## What?

```bash
#
# Switch to kustomize v2.1.0
#
╰─» asdf global kustomize v2.1.0
    kustomize version
    kustomize build kustomize/instances/gasp/

Version: {KustomizeVersion:2.1.0 GitCommit:af67c893d87c5fb8200f8a3edac7fdafd61ec0bd BuildDate:2019-06-18T22:01:59Z GoOs:linux GoArch:amd64}

apiVersion: v1
kind: Namespace
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/gasp
    nodevops.io/kustomize-component: atlantis
  name: atlantis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/gasp
    nodevops.io/kustomize-component: atlantis
    nodevops.io/owner: techops
  labels:
    app: atlantis
  name: atlantis
  namespace: atlantis
.
.
.
      volumes:
      - configMap:
          defaultMode: 511
          name: atlantis-custom-entrypoint
        name: atlantis-custom-entrypoint
      - name: atlantis-data
        persistentVolumeClaim:
          claimName: atlantis-data
      - emptyDir:
          medium: Memory
        name: atlantis-github-ssh
      - configMap:
          name: consul-template-config
        name: consul-template-config
      - emptyDir:
          medium: Memory
        name: shared-data
      - emptyDir:
          medium: Memory
        name: vault-token
      - configMap:
          name: vault-agent-config
        name: vault-agent-config

#
# Switch to kustomize v3.0.0
#
╰─» asdf global kustomize v3.0.0
    kustomize version
    kustomize build kustomize/instances/gasp/

Version: {KustomizeVersion:3.0.0 GitCommit:e0bac6ad192f33d993f11206e24f6cda1d04c4ec BuildDate:2019-07-03T18:21:24Z GoOs:linux GoArch:amd64}

Error: no matches for OriginalId apps_v1_Deployment|~X|atlantis; no matches for CurrentId apps_v1_Deployment|~X|atlantis; failed to find unique target for patch apps_v1_Deployment|atlantis

```

## The culprit

in `kustomize/components/atlantis/resources/deployment.yml`, remove the namespace declaration (that is superfluous as the namespace is specified in `kustomization.yml` but that should not fail)

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: atlantis
  namespace: atlantis
  labels:
    app: atlantis
  annotations:
    nodevops.io/owner: techops
.
.
.
```

and it works for both `v2.1.0` and `v3.0.0`

## Other fix

Specify the namespace in the patch target like suggested in <https://github.com/kubernetes-sigs/kustomize/issues/1332>

```yaml
patchesJson6902:
  # atlantis
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: atlantis
      namespace: atlantis
    path: patches/atlantis/atlantis--deployment.yml
```

```bash
#
# Switch to kustomize v3.0.0
#
╰─» asdf global kustomize v3.0.0
    kustomize version
    kustomize build kustomize/instances/gasp/

Version: {KustomizeVersion:3.0.0 GitCommit:e0bac6ad192f33d993f11206e24f6cda1d04c4ec BuildDate:2019-07-03T18:21:24Z GoOs:linux GoArch:amd64}
Version: {KustomizeVersion:3.0.0 GitCommit:e0bac6ad192f33d993f11206e24f6cda1d04c4ec BuildDate:2019-07-03T18:21:24Z GoOs:linux GoArch:amd64}
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/works
    nodevops.io/kustomize-component: atlantis
  name: atlantis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/works
    nodevops.io/kustomize-component: atlantis
    nodevops.io/owner: techops
  labels:
    app: atlantis
  name: atlantis
  namespace: atlantis
.
.
.
```
