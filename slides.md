---
title: Kubernetes Volumes
author: Andrew Farries
date: 2021-02-28
extensions: []
styles: {}
---
# Kubernetes Volumes

Let's learn about data persistence in Kubernetes.

We will cover:

Persistence tied to the lifetime of a Pod:
* `Volumes`

Peristence independent of Pod lifetime:
* `PersistentVolumes`
* `PersistentVolumeClaims`
* `StorageClasses`

The new snapshot related functionality just of beta in Kubernetes 1.20:
* `VolumeSnapshotContents`
* `VolumeSnapshots`
* `VolumeSnapshotClasses`

---
# Prerequisites

Minikube set up to allow persistent volumes and volume snapshots.

Ensure that we have run:

```bash
> minikube addons enable volumesnapshots

```

and:

```bash
> minikube addons enable csi-hostpath-driver
```

---
# Volumes

The lifetime of a volume (as opposed to a `PersistentVolume` that we will cover later) is tied to the lifetime of the `Pod` in which it is declared.

A volume is not an independent Kubernetes resource type. It exists only as a field in a `Pod` spec.

`Volumes` are most often used to:
* Mount an `emptyDir` volume into a pod to act as a shared "scratch" area for multiple containers.
* Mount a `ConfigMap` or a `Secret` into a `Pod`.

Create a Pod with two containers and a shared `emptyDir` volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: bbox
  name: bbox
spec:
  volumes:
  - name: myvol
    emptyDir: {}
  containers:
  - name: bbox
    args:
    - sleep
    - "3600"
    image: busybox
    volumeMounts:
    - name: myvol
      mountPath: /stuff
  - name: bbox2
    args:
    - sleep
    - "3600"
    image: busybox
    volumeMounts:
    - name: myvol
      mountPath: /stuff
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

---
# Static Provisioning vs Dynamic Provisioning

`PersistentVolumes` can be *statically provisioned* or *dynamically provisioned*.

Static provisioning means that the cluster administrator creates a set of `PersistentVolume` resources to be used by applications in the cluster. Pods have to use one of these resources to get access to a `PersistentVolume` - there is no other way to get hold of a `PersistentVolume`.

Dynamic provisioning allows new `PersistentVolumes` to be created 'on demand' by requesting one from a `StorageClass`. The set of `PersistentVolumes` is therefore not fixed ahead of time by the cluster administrator.

---
# PersistentVolumes

We can create a new `PersistentVolume` in the cluster as follows:

```yaml
foo: bar
```

---
# PersistentVolumeClaims

`PersistentVolumeClaims` (`or just PVCs`)

---
# Dynamic PVC provisioning with StorageClasses

New `PersistentVolumes` can be created dynamically by a PVC by requesting a new one from a `StorageClass` rather than binding to an existing one.

> StorageClasses are machines for creating PVCs on demand.

[here](www.google.com)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: rook-ceph-block
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

Create this PVC and then run:

```
kubectl get pvc,pv
```

You can see that a new `PersistentVolume` has been created 'on demand' by the `StorageClass` and bound to the `my-pvc` PVC.

---
# Snapshots

Volume snapshots were introduced in alpha form in Kubernetes X.YZ, in beta in Kubernetes X.YZ and came of beta only very recently in Kubernetes X.YZ.

---
# Volume Cloning

In addition to creating a new `PVC` by specifiying a `VolumeSnapshot` in the `dataSource` field, it is also possible to "clone" an existing `PVC` by specifying another `PVC` in the `dataSource` field. This effectively removes the need to snapshot a `PVC` before cloning it.

*Not all CSI implementations support this feature!*

---
# Resources

