# kubelogin
- To login to remote kubernetese cluster from local development machine. 
### Create client certificate for new user called "dev" 

login as root on kubernetes api server 
```
openssl genrsa -out dev.key 2048
openssl req -new -key dev.key -out dev.csr -subj "/CN=dev/O=local"
openssl x509 -req -in dev.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out dev.crt -days 500
```
Locate your ca.key and ca.crt files on kube API server path /etc/kubernetes/pki/ path.

3 new files with name dev.csr dev.key and dev.crt will be created

On local development machine create kubernetes credential, context, and cluster using dev.key and dev.crt and ca.key and ca.crt files 

```
kubectl config set-credentials dev --client-certificate=dev.crt  --client-key=dev.key
kubectl config set-cluster dev --embed-certs=true --server=https://10.0.0.58:6443 --certificate-authority=ca.crt
kubectl config set-context dev --cluster=dev --namespace=dev --user=dev
``` 
--server=https://10.0.0.58:6443 is load balancer ip and port for kubernetes api servers.

To verify the access run below command it will throw error for role creation 

```
kubectl --context=dev get pods
Error from server (Forbidden): pods is forbidden: User "dev" cannot list pods in the namespace "dev"
```

Now on API server, create deployment role using role-deployment-manager.yaml and bind it with dev user in dev namespace. 

```
kubectl create namespace dev

---
  kind: Role
  apiVersion: rbac.authorization.k8s.io/v1beta1
  metadata:
    namespace: dev
    name: deployment-manager
  rules:
  - apiGroups: ["", "extensions", "apps"]
    resources: ["deployments", "replicasets", "pods"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"] # You can also use ["*"]
---
  kind: RoleBinding
  apiVersion: rbac.authorization.k8s.io/v1beta1
  metadata:
    name: deployment-manager-binding
    namespace: dev
  subjects:
  - kind: User
    name: dev
    apiGroup: ""
  roleRef:
    kind: Role
    name: deployment-manager
    apiGroup: ""    

Now verify access on local machine again.

```
kubectl --context=dev get pods
No resources found
```
