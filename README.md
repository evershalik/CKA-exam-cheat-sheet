# CKA Exam Cheatsheet

A comprehensive kubectl command reference for the Certified Kubernetes Administrator (CKA) exam.

## Core Commands

### Pods
```bash
# Create and manage pods
kubectl run nginx --image=nginx
kubectl get po --show-labels
kubectl get po --field-selector status.phase=Running -A
kubectl run redis --image=redis:alpine -l='tier=db'
kubectl run webapp --image=kodekloud/webapp-color --env="APP_COLOR=green"
kubectl run static-busybox --restart=Never --image=busybox --dry-run=client -o yaml --command -- sleep 1000 

# Generate YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl run webapp --image=kodekloud/webapp-color --dry-run=client -o yaml -- --color green
```

### Deployments
```bash
# Create deployments
kubectl create deployment nginx --image=nginx --replicas=4
kubectl create deploy redis-deploy --image=redis --replicas=2 -n dev-ns

# Manage deployments
kubectl set image deployment/nginx nginx=nginx:1.15 --record
kubectl scale deployment nginx --replicas=5
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10  # hpa
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
```

### Services
```bash
# Expose services
kubectl expose pod redis --port=6379 --name redis-service
kubectl expose deploy nginx --name=front-end-svc --type=NodePort --port=80 --target-port=80 --protocol=TCP
kubectl create service clusterip redis --tcp=6379:6378
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080
```

### Ingress
```bash
# Create ingress resources
kubectl create ingress webapp --rule="app.example.com/=webapp:80"
kubectl create ingress api --rule="/api=api-service:8080" --dry-run=client -o yaml
```

## Configuration Management

### ConfigMaps & Secrets
```bash
# ConfigMaps
kubectl create configmap app-config --from-literal=APP_COLOR=blue
kubectl describe cm app-config
kubectl config view

# Secrets
kubectl create secret generic db-secret --from-literal=DB_HOST=mysql
```

### Labels & Selectors
```bash
# Filter by labels
kubectl get pods --selector env=dev
kubectl get pods --selector='bu=finance'
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

## Node Management

### Taints & Tolerations
```bash
# Apply/remove taints
kubectl taint nodes node01 spray=mortein:NoSchedule
kubectl taint nodes node01 spray=mortein:NoSchedule-

# Node labels
kubectl label node node01 size=Large
```

### Node Maintenance
```bash
# Drain and cordon nodes
kubectl drain node01 --ignore-daemonsets --force
kubectl cordon node01
kubectl uncordon node01
```
### Upgrading kubeadm clusters
```bash
sudo apt update
sudo apt-cache madison kubeadm
kubectl drain <master-node-name> --ignore-daemonsets
sudo apt-get install -y kubeadm=1.22.2-00 kubelet=1.22.2-00 kubectl=1.22.2-00
kubeadm upgrade plan
kubeadm upgrade apply v1.22.2
sudo systemctl daemon-reload
sudo systemctl restart kubelet
kubectl uncordon <master-node-name>
```

## Cluster Management

### ETCD Backup & Restore
```bash
# Backup
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/snapshot.db

# Restore
ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot.db --data-dir /var/lib/etcd-backup
```

### Certificates
```bash
# Certificate operations
kubectl certificate approve user-csr
kubectl get csr
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

### Enable and Start cri-docker service
```bash
sudo systemctl enable cri-docker.service
sudo systemctl enable cri-docker.socket
sudo systemctl start cri-docker.service
sudo systemctl start cri-docker.socket
```


## Security & RBAC

### Authentication & Authorization
```bash
# Check permissions
kubectl auth can-i create deployments
kubectl auth can-i create pods --as dev-user

# Create roles and bindings
kubectl create role developer --verb=create,get,list,delete --resource=pods
kubectl create rolebinding dev-binding --role=developer --user=dev-user
kubectl create clusterrole nodes-admin --verb=create,list,delete --resource=nodes
kubectl create clusterrolebinding nodes-binding --clusterrole=nodes-admin --serviceaccount=<namespace>:<name> --user=admin
```

### Service Accounts
```bash
# Service account operations
kubectl create sa dashboard-sa
kubectl create token dashboard-sa
```

### Contexts
```bash
# Context management
kubectl config get-contexts
kubectl config current-context
kubectl config use-context production
```

## Storage

### Persistent Volumes
```bash
# PV and PVC operations
kubectl get pv,pvc
kubectl describe pvc local-pvc
```

## Monitoring & Troubleshooting

### Resource Monitoring
```bash
# Resource usage
kubectl top nodes
kubectl top pods --containers=true
kubectl top pods --sort-by=cpu
kubectl cluster-info
```

### Logs & Debugging
```bash
# Pod logs
kubectl logs -f pod-name
kubectl logs pod-name -c container-name
kubectl logs -p pod-name  # Previous instance

# Debug pods
kubectl exec -it pod-name -- /bin/bash
kubectl describe pod pod-name
```


### Troubleshooting Commands
```bash
# Check cluster components
kubectl get componentstatuses
kubectl get events --sort-by=.metadata.creationTimestamp

# Service debugging
kubectl get svc,ep
kubectl exec -it pod-name -- nslookup service-name
```

### Network Troubleshooting
```bash
# DNS resolution
kubectl exec -it pod-name -- nslookup kubernetes.default
kubectl run dns-test --image=busybox:1.28 --rm -it --restart=Never -- nslookup service-name

# Network connectivity
kubectl run netshoot --image=nicolaka/netshoot --rm -it --restart=Never -- nc -zv service-name 80
```

## Useful Outputs

### Custom Columns
```bash
# Custom output formats
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
kubectl get nodes -o custom-columns=NAME:.metadata.name,CPU:.status.capacity.cpu
kubectl get deployments -o custom-columns=NAME:.metadata.name,READY:.status.readyReplicas
```

### JSONPath
```bash
# JSONPath examples
kubectl get nodes -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'
```

## Static Pods

```bash
# Static pod management
ls /etc/kubernetes/manifests/
kubectl run static-pod --image=nginx --dry-run=client -o yaml > /etc/kubernetes/manifests/static-pod.yaml
```

## Quick Reference

### Important Paths
- kubelet config: `/var/lib/kubelet/config.yaml`
- Static pods: `/etc/kubernetes/manifests/`
- PKI certificates: `/etc/kubernetes/pki/`
- CNI plugins: `/opt/cni/bin/`
- CNI config: `/etc/cni/net.d/`

### Common Flags
- `--dry-run=client -o yaml`: Generate YAML without creating
- `--force --grace-period=0`: Force delete resources
- `--all-namespaces` or `-A`: All namespaces
- `--sort-by=.metadata.creationTimestamp`: Sort output
- `--no-headers`: Remove headers from output

## Resources

- [Official Kubernetes Documentation](https://kubernetes.io/docs/)
- [CKA Exam Curriculum](https://github.com/cncf/curriculum)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

**Exam Tips:**
- Practice with `kubectl` completion enabled
- Use `--dry-run=client -o yaml` to generate templates
- Master JSONPath for custom outputs
- Remember to check all namespaces with `-A`
- Use `kubectl explain` for resource specifications

## Useful Links:
> **NOTE**: _Must have a look before your CKA_

Github
- https://github.com/ascode-com/wiki/blob/main/certified-kubernetes-administrator/README.md
- https://gist.github.com/Nurlan199206/d4cd11487f2ffd8ede01085dced3a430

Videos
- https://youtu.be/8VK9NJ3pObU

Sample Questions for CKA:
- https://www.itexams.com/exam/CKA
- https://youtu.be/eGv6iPWQKyo?si=v1lE2QsxeD8LNWgB
- https://www.youtube.com/watch?v=LkPOMceDTZY&list=PLYEK_dHOjwtNoS_SYcyntJ8hyhOI-Px8T
- https://www.youtube.com/watch?v=rNgTxL5Iqxo&list=PLFAJ6I3pCvzouqwyaIGuu2tzmY_bz8et9
