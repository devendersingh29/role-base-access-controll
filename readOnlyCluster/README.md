# Kubernetes ClusterRBAC Setup — `max` Service Account (rbac Namespace)

## Prerequisites
* Kubernetes Cluster
* kubectl configured
* `rbac` namespace already created

---

## 1. Create Namespace

```bash
kubectl create namespace rbac
```

Verify:

```bash
kubectl get ns
```

---

## 2. Apply RBAC Manifest

Apply the `rbac.yaml` file to create ServiceAccount, ClusterRole and ClusterRoleBinding:

```bash
kubectl apply -f rbac.yaml
```

Verify ServiceAccount:

```bash
kubectl get serviceaccount max -n rbac
```

Verify ClusterRole:

```bash
kubectl get clusterrole read-only-cluster
```

Verify ClusterRoleBinding:

```bash
kubectl get clusterrolebinding max-read-binding
```

---

## 3. Generate Token

Token generate karo `max` ServiceAccount ke liye:

```bash
kubectl create token max -n rbac
```

> ⚠️ Token copy kar lo — next step mein config file mein paste karna hai.

---

## 4. Add Token to Kubeconfig

`max-config.yaml` file mein `token:` field mein Step 3 ka token paste karo:

```yaml
users:
- name: max
  user:
    token: <YAHAN-TOKEN-PASTE-KARO>
```

CA certificate data lene ke liye:

```bash
kubectl config view --raw
```

---

## 5. Verify Access

Pods list karo scoped kubeconfig se:

```bash
kubectl --kubeconfig=max-config.yaml get pods -n rbac
```

All namespaces mein pods dekho (ClusterRole hai toh allowed hai):

```bash
kubectl --kubeconfig=max-config.yaml get pods -A
```

All resources dekho:

```bash
kubectl --kubeconfig=max-config.yaml get all -A
```

---

## 6. Confirm Denied Access

ClusterRole sirf read-only hai — write operations blocked hain:

```bash
# Forbidden aana chahiye
kubectl --kubeconfig=max-config.yaml delete pod <pod-name> -n rbac
kubectl --kubeconfig=max-config.yaml create deployment test --image=nginx -n rbac
```

---

## Permissions Summary

| Resource | get | list | watch | create | delete |
|---|:---:|:---:|:---:|:---:|:---:|
| All resources (all namespaces) | ✅ | ✅ | ✅ | ❌ | ❌ |
| Cross-namespace read | ✅ | ✅ | ✅ | ❌ | ❌ |
| Write / Delete | ❌ | ❌ | ❌ | ❌ | ❌ |

> **Note:** `max` ke paas **ClusterRole** hai — iska matlab poore cluster mein read-only access hai, sirf ek namespace tak limited nahi.

---

## Useful Commands

```bash
kubectl get serviceaccount max -n rbac
kubectl get clusterrole read-only-cluster
kubectl get clusterrolebinding max-read-binding
kubectl describe clusterrole read-only-cluster
kubectl describe clusterrolebinding max-read-binding
kubectl get all -n rbac
kubectl get events -n rbac
```

Check permissions:

```bash
kubectl auth can-i --list \
  --as=system:serviceaccount:rbac:max
```
