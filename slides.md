---
title: Kubernetes Volumes
author: Andrew Farries
date: 2021-02-28
extensions: []
styles: {}
---
# Kubernetes Volumes

Let's learn about how Kubernetes handles persistent data.

* `Volumes`
* `PersistentVolumes`
* `PersistentVolumeClaims`
* `StorageClasses`
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
* Mount a `ConfigMap` or a `Secret` into a `Pod`

---
# Static Provisioning vs Dynamic Provisioning

`PersistentVolumes` can be *statically provisioned* or *dynamically provisioned*.

Dynamic provisioning is most relevant for Spawn, so we will cover static provisioning only briefly here.

---
# PersistentVolumes
