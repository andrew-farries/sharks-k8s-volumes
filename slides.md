---
title: Kubernetes Volumes
author: Andrew Farries
date: 2021-02-28
extensions: []
styles: {}
---
# Kubernetes Volumes

Let's learn about volumes and data persistence in Kubernetes.

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

Create a new `minikube` namespace in which to work:

```
kubectl create ns volumes-onboarding
```

Set the current context to the new namespace:

```
kubectl config set-context --current --namespace volumes-onboarding
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
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  volumes:  # This is where the volumes for the pod are defined.
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
    volumeMounts:  # This is where the volume is mounted into the pod.
    - name: myvol
      mountPath: /data
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Now run:

```
kubectl exec -it busybox -c bbox -- /bin/bash
```

and create a file with some dummy data in the `/stuff` directory. Exit the shell and run:

```
kubectl exec -it busybox -c bbox2 -- /bin/bash
```

Look in the `/data` directory and see that the data you wrote in the first container is also present in the other container. This is how two containers in a `Pod` can share a 'scratch area' on disk.

Delete the pod:

```
kubectl delete pod busybox
```

Once the `Pod` is deleted, the volume ceases to exist.

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
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
```

Then create a `PVC` to claim the volume:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: ""
  volumeName: pv0001
  resources:
    requests:
      storage: 5Gi
  accessModes:
    - ReadWriteOnce
```

Once the claim is bound, the `PVC` can be mounted into a Pod by specifying the PVC in the `volumes` section of the Pod spec:

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
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - args:
    - sleep
    - "3600"
    image: busybox
    name: bbox
    volumeMounts:
    - name: myvol
      mountPath: /stuff
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

As before `k exec` into the Pod and write some data under the volume's mountpoint, then exit and delete the Pod.

Notice that the `PVC` still exists in the cluster - its lifetime is independent of the `Pod`.

Recreate another `Pod` using the same `Pod` spec and confirm that the data is still there.

Note: `PersistentVolumeClaims` can also be bound by specifying a label selector, rather than a PV name.

---
# Why PersistentVolumeClaims?

`PersistentVolumeClaims` are supposed to abstract away the details of where a particuar `PersistentVolume` comes from. When deploying a pod, you don't care about where the storage comes from, you just want '5Gi' of storage.

This goes back to the idea that the cluster administrator is a separate person/group from the people deploying and running workloads on the cluster. Statically provisioned `PersistentVolumes` are the concern of the cluster admins. `PersistentVolumeClaims` are the concern of the cluster users.

---
# Dynamic PVC provisioning with StorageClasses

New `PersistentVolumes` can be created dynamically by a PVC by requesting a new one from a `StorageClass` rather than binding to an existing one.

See what `StorageClasses` are available with

```
kubectl get sc
```

Use the `rook-ceph-block` `StorageClass` to create a new `PVC` that requests a new `PersistentVolume` from that `StorageClass`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc2
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

As before this `PVC` can be mounted into a pod:

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
    persistentVolumeClaim:
      claimName: my-pvc2
  containers:
  - args:
    - sleep
    - "3600"
    image: busybox
    name: bbox
    volumeMounts:
    - name: myvol
      mountPath: /stuff
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

`k exec -it bbox -- /bin/bash`, write some data, exit and delete the `Pod`. The dynamically provisioned `PVC` `my-pvc2` is still present in the cluster.

---
# Snapshots

The `VolumeSnapshots` feature went GA in Kubernetes 1.20. This allows PVCs to be snapshotted, and then new PVCs can be created based on that snapshot.

`VolumeSnapshots` are resources in the cluster just like `Pods`, `ReplicaSets`, and so on. Create a `VolumeSnapshot` by specifying the PVC that is to be snapshotted:

Let's snapshot the `rook-ceph-block` PVC we created in the previous step:

```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  volumeSnapshotClassName: csi-rbdplugin-snapclass
  source:
    persistentVolumeClaimName: my-pvc2
```

Now do:

```
kubectl get volumesnapshots
```

We have one new `VolumeSnapshot` created in the cluster.

Connect to the pod that has `my-pvc2` mounted, and write some more data under the mountpoint. This new data will not be included in the `VolumeSnapshot` we just took.

Now create a new `PVC` using the `VolumeSnapshot` as the data source - this gives a new `PVC` based on the `VolumeSnapshot`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-based-on-snapshot
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: rook-ceph-block
  resources:
    requests:
      storage: 50Gi
  dataSource:  # This is the interesting bit.
    name: my-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

`kubectl get pvc` now shows the new PVC. We can mount this into a pod as before:

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
    persistentVolumeClaim:
      claimName: pvc-based-on-snapshot
  containers:
  - args:
    - sleep
    - "3600"
    image: busybox
    name: bbox
    volumeMounts:
    - name: myvol
      mountPath: /stuff
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

Exec into the pod as before and look at the data under `/stuff`. The data does not included the parts that were written after the snapshot was taken.
