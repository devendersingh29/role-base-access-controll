# Kubernetes RBAC Setup — `sunjay` Service Account (Jenkins Namespace)

## Prerequisites
* Kubernetes Cluster
* kubectl configured
* `jenkins` namespace already created

---

## 1. Create Namespace

```bash
kubectl create namespace jenkins
```

Verify:

```bash
kubectl get ns
```

---

## 2. Apply RBAC Manifest

Apply the `rbac.yaml` file to create ServiceAccount, Role and RoleBinding:

```bash
kubectl apply -f rbac.yaml
```

Verify ServiceAccount:

```bash
kubectl get serviceaccount sunjay -n jenkins
```

Verify Role:

```bash
kubectl get role jenkins-role -n jenkins
```

Verify RoleBinding:

```bash
kubectl get rolebinding sunjay-binding -n jenkins
```

---

## 3. Generate Token

Token generate karo `sunjay` ServiceAccount ke liye:

```bash
kubectl create token sunjay -n jenkins
```

> ⚠️ Token copy kar lo — next step mein config file mein paste karna hai.

---

## 4. Add Token to Kubeconfig

`sunjay-config.yaml` file mein `token:` field mein Step 3 ka token paste karo:

```yaml
users:
- name: sunjay
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
kubectl --kubeconfig=sunjay-config.yaml get pods -n jenkins
```

---

## 6. Confirm Denied Access

```bash
# Forbidden aana chahiye
kubectl --kubeconfig=sunjay-config.yaml get deployments -n jenkins
kubectl --kubeconfig=sunjay-config.yaml get secrets -n jenkins

# Other namespace bhi blocked hai
kubectl --kubeconfig=sunjay-config.yaml get pods -n default
```

---

## Permissions Summary

| Resource | get | list | watch | create | delete |
|---|:---:|:---:|:---:|:---:|:---:|
| `pods` (jenkins ns) | ✅ | ✅ | ✅ | ❌ | ❌ |
| Any other resource | ❌ | ❌ | ❌ | ❌ | ❌ |
| Other namespaces | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Useful Commands

```bash
kubectl get serviceaccount -n jenkins
kubectl get role -n jenkins
kubectl get rolebinding -n jenkins
kubectl get all -n jenkins
kubectl describe role jenkins-role -n jenkins
kubectl describe rolebinding sunjay-binding -n jenkins
```

Check permissions:

```bash
kubectl auth can-i --list \
  --as=system:serviceaccount:jenkins:sunjay \
  -n jenkins
```
