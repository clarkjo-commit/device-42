# Device42 Kubernetes Integration - ServiceAccount & Token Setup

This repository outlines the steps required to create a Kubernetes `ServiceAccount`, bind it with cluster-wide read-only access, and generate a long-lived bearer token for use by **Device42** to monitor all namespaces and applications.

---

## 📁 Files Included

- `d42-serviceaccount.yaml`: Creates the `device42-reader` ServiceAccount in the `d42` namespace.
- `d42-clusterrolebinding.yaml`: Binds the `device42-reader` ServiceAccount to the `view` ClusterRole across the entire cluster.
- `d42-token-secret.yaml`: Creates a persistent bearer token Secret tied to the `device42-reader` ServiceAccount.

---

## 🛠️ Step-by-Step Instructions

### 1. Create the Namespace (if it doesn't exist)
```bash
kubectl create namespace d42
```

### 2. Apply the ServiceAccount
This ServiceAccount will be used by Device42 to access the Kubernetes API.
```bash
kubectl apply -f d42-serviceaccount.yaml
```

### 3. Apply the ClusterRoleBinding
This gives the ServiceAccount cluster-wide read-only access.
```bash
kubectl apply -f d42-clusterrolebinding.yaml
```

### 4. Create a Long-Lived Bearer Token
Apply the following secret manifest to generate a service account token that does not expire:
```bash
kubectl apply -f d42-token-secret.yaml
```

After a few seconds, retrieve the token:
```bash
kubectl -n d42 get secret device42-token -o jsonpath='{.data.token}' | base64 -d
```

You can now use this token as a bearer token in the `Authorization` header:
```
Authorization: Bearer <your-token>
```

---

## 🔐 Security Notes
- The `view` ClusterRole allows read-only access to most Kubernetes resources across all namespaces.
- This token does **not expire** and should be secured appropriately.
- For enhanced security, consider rotating this token periodically and limiting access via a custom ClusterRole.

---

## 🧪 Verify Permissions
You can test the access granted to the token:
```bash
kubectl auth can-i list pods --as=system:serviceaccount:d42:device42-reader --all-namespaces
```

Expected output:
```
yes
```

---

## 📚 Reference
- Kubernetes ServiceAccount Docs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
- Kubernetes RBAC Docs: https://kubernetes.io/docs/reference/access-authn-authz/rbac/