# Troubleshooting Kubernetes StatefulSet and Persistent Volume Issues

## Problem:

- A StatefulSet application  (e.g., a database or web service) works fine on AWS EKS but fails on AKS, GKE, or Minikube.
- Pods fail to start and remain in a `Pending` state due to the error `Pod has UnboundImmediatePersistentVolumeClaims`.

## Root Cause:

- The StatefulSet is requesting a Persistent Volume (PV) using a PersistentVolumeClaim (PVC), but Kubernetes cannot satisfy the claim because the storage class is incompatible with the new environment.
- The `storageClassName` in the StatefulSet YAML file is set to `ebs`, which is specific to AWS EKS.

## StatefulSets vs Deployments:

A StatefulSet is a Kubernetes workload API object designed for stateful applications (e.g., databases like MySQL, MongoDB, or services that need stable network identities and persistent storage).

- In a StatefulSet, pods are created sequentially, not simultaneously like deployments.
- This sequential creation ensures data consistency and prevents failures in dependent pods.
- Pods are tightly coupled with Persistent Volumes (PVs) for data persistence.
- Pod-1 must succeed before Pod-2 can start, and so on.
- If Pod-1 fails, Pod-2 will not even attempt to start.

## Persistent Volumes (PV) and Persistent Volume Claims (PVC)

**Persistent Volume (PV)**
- A PV is a piece of storage provisioned in the cluster.
- It can be provisioned statically (pre-created) or dynamically (created on demand by a StorageClass).

**Persistent Volume Claim (PVC)**
- A PVC is a request for storage by a pod.
- It specifies the size (e.g., 1GB) and storage class (e.g., ebs, standard).

**StorageClass**
- A StorageClass defines how storage volumes are dynamically provisioned.
- Different Kubernetes environments have different default storage classes:
    - AWS EKS: ebs
    - Azure AKS: managed-premium
    - GKE: standard-rwo
    - Minikube: standard

## Solution Steps:

```bash
# Identify the Available Storage Classes in your target cluster (for example: Minikube):
kubectl get storageclass

# Update the StatefulSet YAML file:
# In the PersistentVolumeClaim section of your StatefulSet YAML, update the storageClassName to match the target cluster:
storageClassName: standard

# Delete Stale PVCs (If Required)
# Sometimes, PVCs and PVs from previous failed attempts might still exist. Delete them manually:
kubectl get pvc
kubectl delete pvc <pvc-name>

# Delete and reapply the Updated StatefulSet YAML Manifest
kubectl delete -f sample-statefulset.yaml
kubectl apply -f sample-statefulset.yaml

# Verify Pod Status
kubectl get pods -w

# Pods should now start one by one in a sequential order.
```

## Using CSI (Container Storage Interface) Drivers:

- CSI drivers allow Kubernetes to use external storage solutions (e.g., NetApp, Dell EMC) to create Persistent Volumes.
- If the storage you need is not native to the cloud provider (e.g., NetApp on AWS), Kubernetes uses a CSI Driver.

```bash
# Install the CSI driver provided by your storage vendor using Helm or Kubernetes operators.
helm install netapp-csi-driver ...

# Modify your StatefulSet YAML to use the appropriate storageClassName provided by the CSI driver:
storageClassName: netapp-storage-class

Apply the StatefulSet and monitor the pods.
```
