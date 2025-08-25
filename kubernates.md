### kubectl config get-contexts
 - Check which cluster its pointing to
 - if minicube local
 - If it’s my-remote, then it’s pointing to the remote cluster.
 - CURRENT   NAME           CLUSTER        AUTHINFO       NAMESPACE
          minikube       minikube       minikube       default
          my-remote      my-remote      user-remote    default
Ah 👍 if you want to remove the **entire `preview` namespace itself** (not just the pods inside), you can do it like this:

---

### 🔥 Delete the namespace (removes everything inside it)

```bash
kubectl delete namespace preview
```

That will delete:

* All pods
* Deployments
* Services
* ConfigMaps
* Secrets
* PVCs
* etc.

Basically everything scoped to that namespace.

---

