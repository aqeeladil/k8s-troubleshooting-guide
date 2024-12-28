# Scheduling and Troubleshooting Pods.

In Kubernetes, the scheduler is responsible for assigning pods to nodes in the cluster based on various criteria. Sometimes, you might encounter situations where pods are not being scheduled as expected. This can happen due to factors such as node constraints, pod requirements, or cluster configurations.

## 1. Node Selector

- Node Selector ensures that a pod is scheduled only on specific nodes in a Kubernetes cluster.
- It allows you to specify a set of key-value pairs that must match the node's labels for a pod to be scheduled on that node.
- Usage: Include a nodeSelector field in the pod's YAML definition to specify the required labels.
- Label Nodes: Use labels to mark nodes with specific characteristics or purposes.
  `kubectl label node <node-name> hardwareType=GPU`
- Pod Specification: Use the nodeSelector field in your pod or deployment YAML to match those labels.
  ```yaml
  spec:
    nodeSelector:
      hardwareType: GPU
  ```
- If no node matches the selector, the pod remains in a Pending state.
- This is a hard constraint, meaning the scheduler will never bypass the rule.

**Use Cases:**
- Hardware Constraints: Workloads requiring specialized hardware like GPUs or ARM processors.
  - A machine learning model that needs to run on a GPU node:
  ```yaml
  spec:
    nodeSelector:
      hardwareType: GPU
  ```
- Environment Isolation:
  - Separate workloads by environment, e.g., staging and production.
  - Label nodes with environment=production or environment=staging, and use nodeSelector in workloads to control placement.

## 2. Node Affinity

- Offers more flexibility compared to nodeSelector by allowing both hard and soft constraints.
- It allows you to specify rules that apply only if certain conditions are met.
- Types:
  - RequiredDuringSchedulingIgnoredDuringExecution: 
    - Similar to nodeSelector (hard constraint).
  - PreferredDuringSchedulingIgnoredDuringExecution: 
    - Scheduler will prioritize preferred nodes but can still place the pod on other nodes if necessary.
- Use Case: Ideal for scenarios where you want pods to prefer certain nodes but still run on others if necessary.
- Example: Prefer nodes with ssd-disk but still run elsewhere if none are available.

## 3. Taints

- Prevents pods from being scheduled on a node unless the pod explicitly tolerates the taint.
- Taints are applied to nodes to repel certain pods. They allow nodes to refuse pods unless the pods have a matching toleration.
- Effects:
  - NoSchedule: Pods without matching tolerations are not scheduled on the node.
  - PreferNoSchedule: Scheduler avoids the node unless no other option is available.
  - NoExecute: Pods already on the node are evicted.
- Use Case: Commonly used during maintenance or when a node should only run specific workloads.
- Usage: Use `kubectl taint nodes <node-name> key=value:effect` to apply taints to nodes. 
- Include tolerations field in the pod's YAML definition to tolerate specific taints.
- `kubectl taint nodes node1 disktype=ssd:NoSchedule`
- Use `kubectl describe node <node-name>` to see applied taints.

**Use Cases:**
- Node Maintenance:
  - Taint a node during an upgrade to ensure no new pods are scheduled:
  `kubectl taint nodes node1 maintenance=true:NoSchedule`
- Workload Isolation:
  - Dedicate nodes to specific workloads by tainting them (e.g., database nodes):
  `kubectl taint nodes node1 role=database:NoSchedule`

## 4. Tolerations

- Tolerations allow pods to "tolerate" a taint and be scheduled on tainted nodes. 
- They override the effect of taints.
- Pods with tolerations matching the taints on a node can run there, even if the node is tainted.
- Use Case: Used for critical pods that must run on nodes with taints.
- Usage: Include tolerations field in the pod's YAML definition to specify which taints the pod tolerates.

**Taints and Tolerations Example:**
- During a rolling upgrade, taint a node to avoid scheduling new pods, but allow critical pods with tolerations to run.

## Troubleshooting Example

1. **Pod Stuck in Pending State:**
  - `kubectl describe pod <pod-name>`
  - Common errors include:
    - "FailedScheduling": Indicates no node satisfies the pod's constraints (e.g., no matching nodeSelector or affinity rules).
    - "Untolerated Taint": Occurs if the node has a taint and the pod lacks a matching toleration.
2. **Verify Node Labels and Taints:**
  - List node labels: `kubectl get nodes --show-labels`
  - Check node taints: `kubectl describe node <node-name>`



