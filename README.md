# Kubernetes Persistent Volumes

Some resources for a sharks onboarding about persistence in K8S.

Cover:
* PersistentVolumes
* PersistentVolumeClaims
* StorageClasses

Then:
* VolumeSnapshotClasses
* VolumeSnapshots
* VolumeSnapshotContents

Might need to be two sessions.

can use lookatme for the presentation.

testing cluster or minikube?
* mini because not everyone has kubectl access to testing.

1. Cover manually creating pvs or just dynamic using storageclasses?
  * how to manually create a pv in minikube?
    * see the docs, can do this for hostpath type pvs.
2. create a pvc by hand
3. Mount it into a pod.
3a. set up a two container pod and mount the same volume into a both. exec into both and write data back and forth.
  * could also demo this with an emptydir volume, note that the lifetime here is the lifetime of the pod.
4. the same but start a pod, write, stop and mount the same pvc into another pod.

have to enable some Minikube addons to get the snapshot stuff working. See here

https://minikube.sigs.k8s.io/docs/tutorials/volume_snapshots_and_csi/

mention access modes.
Also show kubetcl cp to copy stuff in and out of running pods.
mention at the end that we will look at the ceph specific stuff in a different session.
