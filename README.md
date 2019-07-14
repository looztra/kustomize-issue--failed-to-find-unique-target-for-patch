# Kustomization fails with version >= 3.0.0 because namespace

## Pointers

- <https://github.com/kubernetes-sigs/kustomize/issues/1332>
- <https://github.com/kubernetes-sigs/kustomize/issues/1351>

## Usecase 1

- namespace in the _base_ resource specified to *any* value
- namespace specified in the _base_ `kustomization.yml`
- patchesJson6902 in the _overlay_ not specifying a namespace

Kustomize `v2.1.0` : **OK**
Kustomize `v3.0.x` : **NOK**

```bash
#
# Switch to kustomize v2.1.0
#
╰─» asdf global kustomize v2.1.0
    kustomize version
    echo
    kustomize build kustomize/instances/ok-v2.1.0-nok-v3.0.x

Version: {KustomizeVersion:2.1.0 GitCommit:af67c893d87c5fb8200f8a3edac7fdafd61ec0bd BuildDate:2019-06-18T22:01:59Z GoOs:linux GoArch:amd64}

apiVersion: v1
kind: Namespace
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v2.1.0-nok-v3.0.x
    nodevops.io/kustomize-component: atlantis
  name: atlantis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v2.1.0-nok-v3.0.x
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
    echo
    kustomize build kustomize/instances/ok-v2.1.0-nok-v3.0.x

Version: {KustomizeVersion:3.0.0 GitCommit:e0bac6ad192f33d993f11206e24f6cda1d04c4ec BuildDate:2019-07-03T18:21:24Z GoOs:linux GoArch:amd64}

Error: no matches for OriginalId apps_v1_Deployment|~X|atlantis; no matches for CurrentId apps_v1_Deployment|~X|atlantis; failed to find unique target for patch apps_v1_Deployment|atlantis

```

### Fix to make it work with Kustomize `v3.0.x`

- remove the namespace in in the _base_ resource (still works with `v2.1.0`)
- **OR** specify the namespace in the patch target (breaks kustomize `v2.1.0`, see Usecase2)

## Usecase 2

- namespace in the _base_ resource specified to *any* value
- namespace specified in the _base_ `kustomization.yml`
- patchesJson6902 in the _overlay_ specifying the namespace

Kustomize `v2.1.0` : **NOK**
Kustomize `v3.0.x` : **OK**

```bash
#
# Switch to kustomize v2.1.0
#
╰─» asdf global kustomize v2.1.0
    kustomize version
    echo
    kustomize build kustomize/instances/ok-v3.0.x-nok-v2.1.0

Version: {KustomizeVersion:2.1.0 GitCommit:af67c893d87c5fb8200f8a3edac7fdafd61ec0bd BuildDate:2019-06-18T22:01:59Z GoOs:linux GoArch:amd64}

Error: couldn't find target apps_v1_Deployment|atlantis|atlantis for json patch


#
# Switch to kustomize v3.0.0
#
╰─» asdf global kustomize v3.0.0
    kustomize version
    echo
    kustomize build kustomize/instances/ok-v3.0.x-nok-v2.1.0

Version: {KustomizeVersion:3.0.0 GitCommit:e0bac6ad192f33d993f11206e24f6cda1d04c4ec BuildDate:2019-07-03T18:21:24Z GoOs:linux GoArch:amd64}

apiVersion: v1
kind: Namespace
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v3.0.x-nok-v2.1.0
    nodevops.io/kustomize-component: atlantis
  name: atlantis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v3.0.x-nok-v2.1.0
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

## Usecase 3

- **NO** namespace in the _base_ resource
- namespace specified in the _base_ `kustomization.yml`
- patchesJson6902 in the _overlay_ **NOT** specifying the namespace

Kustomize `v2.1.0` : **OK**
Kustomize `v3.0.x` : **OK**

```bash
#
# Switch to kustomize v2.1.0
#
╰─» asdf global kustomize v2.1.0
    kustomize version
    echo
    kustomize build kustomize/instances/ok-v3.0.x-ok-v2.1.0

Version: {KustomizeVersion:2.1.0 GitCommit:af67c893d87c5fb8200f8a3edac7fdafd61ec0bd BuildDate:2019-06-18T22:01:59Z GoOs:linux GoArch:amd64}

apiVersion: v1
kind: Namespace
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v3.0.x-ok-v2.1.0
    nodevops.io/kustomize-component: atlantis
  name: atlantis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v3.0.x-ok-v2.1.0
    nodevops.io/kustomize-component: atlantis
    nodevops.io/owner: techops
  labels:
    app: atlantis
  name: atlantis
  namespace: atlantis
.
.
.

#
# Switch to kustomize v3.0.0
#
╰─» asdf global kustomize v3.0.0
    kustomize version
    echo
    kustomize build kustomize/instances/ok-v3.0.x-nok-v2.1.0

Version: {KustomizeVersion:3.0.0 GitCommit:e0bac6ad192f33d993f11206e24f6cda1d04c4ec BuildDate:2019-07-03T18:21:24Z GoOs:linux GoArch:amd64}

apiVersion: v1
kind: Namespace
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v3.0.x-ok-v2.1.0
    nodevops.io/kustomize-component: atlantis
  name: atlantis
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    nodevops.io/generated-by: kustomize
    nodevops.io/kustomize-assembly: instance/ok-v3.0.x-ok-v2.1.0
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
