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
> minikube addon enable foo
```

and:

```bash
> minikube addon enable bar
```

---
# Volumes

The lifetime of a volume (as opposed to a `PersistentVolume` that we will cover later) is tied to the lifetime of the `Pod` in which it is declared.

A volume is not an independent Kubernetes resource type. It exists only as a field in a `Pod` spec.

`Volumes` are most often used to:
* Mount an `emptyDir` volume into a pod to act as a shared "scratch" area for multiple containers.
* Mount a `ConfigMap` or a `Secret` into a `Pod`.

---
# Static Provisioning vs Dynamic Provisioning

`PersistentVolumes` can be *statically provisioned* or *dynamically provisioned*.

Dynamic provisioning is most relevant for Spawn, so we will cover static provisioning only briefly here.

---
# PersistentVolumes

---
# PersistentVolumeClaims

`PersistentVolumeClaims` (`or just PVCs`)

---
# Dynamic PVC provisioning with StorageClasses

> StorageClasses are machines for creating PVCs on demand.

[here](www.google.com)

---
# Snapshots

Volume snapshots were introduced in alpha form in Kubernetes X.YZ, in beta in Kubernetes X.YZ and came of beta only very recently in Kubernetes X.YZ.

---
# Volume Cloning

In addition to creating a new `PVC` by specifiying a `VolumeSnapshot` in the `dataSource` field, it is also possible to "clone" an existing `PVC` by specifying another `PVC` in the `dataSource` field. This effectively removes the need to snapshot a `PVC` before cloning it.

*Not all CSI implementations support this feature!*

---
# Resources

