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
* `VolumeSnapshots`
* `VolumeSnapshotClasses`

---
# Prerequisites

Minikube set up to allow persistent volumes and volume snapshots.

Ensure that we have run:

```
> minikube delete
```

```
> minikube start
```

```bash
> minikube addons enable csi-hostpath-driver
```

```bash
> minikube addons enable volumesnapshots
```

---
# Volumes

The lifetime of a volume (as opposed to a `PersistentVolume` that we will cover later) is tied to the lifetime of the `Pod` in which it is declared.

A volume is not an independent Kubernetes resource type. It exists only as a field in a `Pod` spec.

`Volumes` are most often used to:
* Mount an `emptyDir` volume into a pod to act as a shared "scratch" area for multiple containers.
* Mount a `ConfigMap` or a `Secret` into a `Pod`.

Create a Pod with two containers and a shared `emptyDir` volume:

```
foo: bar
```

Now run:

```
kubectl exec -it busybox -c bbox -- /bin/bash
```

and create a file with some dummy data in the `/stuff` directory. Exit the shell and run:

```
kubectl exec -it busybox -c bbox2 -- /bin/bash
```

* Look in the `/data` directory.

* Delete the pod

Once deleted, the volume ceases to exist.

---
# Persistent Volumes

`PersistentVolumes` are volumes that have a lifetime independent of any `Pod` in the cluster.

A `PersistentVolume` represents a storage volume on some storage device (like an SSD on the node, or in cloud provider storage like EBS).

Pods get access to `PersistentVolumes` by creating `PersistentVolumeClaims` which bind to the `PersistentVolumes`. Once claimed, a `PersistentVolumeClaim` can be mounted into a Pod.

---
# Static Provisioning vs Dynamic Provisioning

`PersistentVolumes` can be *statically provisioned* or *dynamically provisioned*.

## Static Provisioning

Static provisioning means that the cluster administrator creates a set of `PersistentVolume` resources to be used by applications in the cluster. Pods have to use one of these resources to get access to a `PersistentVolume` - there is no other way to get hold of a `PersistentVolume`.

## Dynamic Provisioning

Dynamic provisioning allows new `PersistentVolumes` to be created 'on demand' by requesting one from a `StorageClass`. The set of `PersistentVolumes` is not fixed ahead of time by the cluster administrator.

---
# Static Provisioning

We can statically provision a new `PersistentVolume` in the cluster as follows:

```yaml
foo: bar
```

Then create a `PVC` to claim the volume:

```yaml
foo:bar
```

Once the claim is bound, the `PVC` can be mounted into a Pod by specifying the PVC in the `volumes` section of the Pod spec:

```yaml
foo: bar
```

As before `k exec` into the Pod and write some data under the volume's mountpoint, then exit and delete the Pod.

Notice that the `PVC` still exists in the cluster - its lifetime is independent of the `Pod`.

Recreate another `Pod` using the same `Pod` spec and confirm that the data is still there.

Note: `PersistentVolumeClaims` can also be bound by specifying a label selector, rather than a PV name.

---
# Why PersistentVolumeClaims?

`PersistentVolumeClaims` are supposed to abstract away the details of where a particuar `PersistentVolume` comes from. When deploying a pod, you don't care about where the storage comes from, you just want '5Gi' of storage.

This basically goes back to the idea that the cluster administrator is a separate person/group from the people deploying and running workloads on the cluster. Statically provisioned `PersistentVolumes` are the concern of the cluster admins. `PersistentVolumeClaims` are the concern of the cluster users.

---
# Dynamic PVC provisioning with StorageClasses

New `PersistentVolumes` can be created dynamically by a PVC by requesting a new one from a `StorageClass` rather than binding to an existing one.

See what `StorageClasses` are available with

```
kubectl get sc
```

> StorageClasses are machines for creating PVCs on demand.

[here](https://www.youtube.com/watch?v=0swOh5C3OVM)

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

You can see that a new `PersistentVolume` has been created 'on demand' by the `StorageClass` and bound to the `my-pvc` PVC. There was no pre-created `PersistentVolume` as there was in the static provisioning case.

---
# Snapshots

Volume snapshots were introduced in alpha form in Kubernetes X.YZ, in beta in Kubernetes X.YZ and came of beta only very recently in Kubernetes X.YZ.

`VolumeSnapshots` are resources in the cluster just like `Pods`, `ReplicaSets`, and so on. Create a `VolumeSnapshot` by specifying the PVC that is to be snapshotted:
