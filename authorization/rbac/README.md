## In Kubernetes, **authz** refers to **authorization** (authz for short), the process of determining whether a request to the Kubernetes API server is permitted based on defined policies and rules.
---
### ### **1. The Authorization Flow**

1. **Authentication (authn)**: The user's identity is verified (e.g., via certificates, tokens).
2. **Authorization (authz)**: Kubernetes checks if the authenticated user has the necessary permissions to perform the requested action.
3. **Admission Controllers**: Further checks or modifications may occur before the request is processed.
---
### **2. Authorization Modes**

The Kubernetes API server supports multiple authorization modes, evaluated in the order they are configured:

#### a. **Role-Based Access Control (RBAC)** _(Most Common)_
- **Role-Based Access Control (RBAC)** in Kubernetes is a method for ==controlling access to resources== based on the roles of users within your cluster. RBAC is highly flexible and allows ==fine-grained access control== for cluster users, applications, and service accounts.
 
- Kubernetes includes a robust RBAC implementation that can be used to segregate users in your cluster. You can set up ==RBAC rules to restrict users== to just the cluster resources they need to access.

## How does RBAC work in Kubernetes?[](https://spacelift.io/blog/kubernetes-rbac#how-does-rbac-work-in-kubernetes)

- The Kubernetes API server has [built-in RBAC support](https://kubernetes.io/docs/reference/access-authn-authz/rbac) that works with both [Users](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#users-in-kubernetes) and [Service Accounts](https://kubernetes.io/docs/concepts/security/service-accounts).

- RBAC is configured in your cluster by enabling the RBAC feature, then creating objects using the resources provided by the ==`rbac.authorization.k8s.io`== API. 

### There are four of these objects available:
#### **1) Role -**
- A Role is a collection of permissions (rules) within a ==**specific namespace**==(Roles are namespaced).
- when you create a Role, you have to ==specify the namespace it belongs in==.
- It defines what actions (verbs) can be performed on what resources.
- **Examples of Verbs**: `get`, `list`, `create`, `update`, `delete`, `watch`.
- Roles are ==typically created based on job functions or business needs==, such as "admin," "developer," or "analyst."
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]                # "" refers to core API group (e.g., pods)
  resources: ["pods"]            # Resource to control
  verbs: ["get", "watch", "list"]  # Allowed actions

```

#### **2) Role Binding**
- A role binding grants the permissions defined in a role to a user or set of users.\
- A RoleBinding grants permissions defined in a **Role** to a user, ==group, or service account== within a specific ==namespace==.
- A RoleBinding grants permissions within a specific namespace whereas a ==ClusterRoleBinding grants that access cluster-wide.==
- A RoleBinding may reference any Role in the same namespace.
- You need to already have a Role named  in that namespace before giving role-ref .
- A RoleBinding ==can also reference a ClusterRole== to grant the permissions defined in that ClusterRole to resources inside the RoleBinding's namespace
- This kind of reference lets you define a set of common roles across your cluster, then reuse them within multiple namespaces.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:                          # Who gets the Role's permissions
- kind: User
  name: alice                      # User name
  apiGroup: rbac.authorization.k8s.io
roleRef:                           # Reference to the Role
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### **3) ClusterRole -**
- Similar to a Role, but **cluster-wide** and can control resources across all namespaces.

- A ClusterRole can be used to grant the same permissions as a Role. Because ==ClusterRoles are cluster-scoped==, you can also use them to grant access to:
     - cluster-scoped resources (like nodes, persistent volumes etc)    
     - non-resource endpoints (like `/healthz`)
     -  namespaced resources (like Pods), across all namespaces    
         For example: you can use a ClusterRole to allow a particular user to run `kubectl get pods --all-namespaces`
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  #"namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
   # at the HTTP level, the name of the resource for accessing Secret
   # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

#### **4) ClusterRole Binding -**
- Similar to a RoleBinding but links a **ClusterRole** to a user, group, or service account across the entire cluster.

```
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding

metadata:
  name: read-secrets-global
subjects:
- kind: Group  #we can have group,user,service account
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```


---
#tip- Rather than referring to individual `resources`, `apiGroups`, and `verbs`, you can use the wildcard `*` symbol to refer to all such objects.
eg.
```

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: example.com-superuser 
rules:
- apiGroups: ["example.com"]
  resources: ["*"]  #gives all resources
  verbs: ["*"]  #gives all verbs
```

---

#### **Verify Permissions**

- Use the `kubectl auth can-i` command to check if the user has the required permissions.
     - `kubectl auth can-i create pods --namespace=dev --as=john`

---

### **6. Debugging and Troubleshooting RBAC**

1) **List Roles and RoleBindings:**

    `kubectl get roles,rolebindings -n <namespace>`
    
2) **List ClusterRoles and ClusterRoleBindings:**
 
    `kubectl get clusterroles,clusterrolebindings`
    
3) **Describe Role/ClusterRole:**

    `kubectl describe role pod-manager -n dev`
    
4) **Check Access:**
    `kubectl auth can-i <verb> <resource> --namespace=<namespace> --as=<user>`

