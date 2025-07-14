- Horizontal pod autoscaling does not apply to objects that can't be scaled (for example: a DaemonSet.)
- It is not possible to directly specify the name of the namespaces in a NetworkPolicy. You must use a namespaceSelector with matchLabels or matchExpressions to select the namespaces based on their labels.
- Choose a CNI That Supports Network Policies

| CNI         | Pod Communication | Network Policies |
| ----------- | ----------------- | ---------------- |
| **Flannel** | ✅ Yes             | ❌ **No**         |
| **Calico**  | ✅ Yes             | ✅ **Yes**        |

- Can You Have Two Default StorageClasses? ❌ No
- Claims can use the volumeName field to explicitly bind to a specific PersistentVolume. You can also leave volumeName unset, indicating that you'd like Kubernetes to set up a new PersistentVolume that matches the claim.
- 

