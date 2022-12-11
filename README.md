# chaos-mesh User Creation, Permession & Token Duration

## User Creation
### 1- viewer User

to create a user with the viewer role apply the example in viewer/rbac.yml : 

```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  namespace: default
  name: example-viewer

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: example-viewer-role
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces"]
  verbs: ["get", "watch", "list"]
- apiGroups: ["chaos-mesh.org"]
  resources: [ "*" ]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: example-viewer-bind
  namespace: default
subjects:
- kind: ServiceAccount
  name: example-viewer
  namespace: default
roleRef:
  kind: Role
  name: example-viewer-role
  apiGroup: rbac.authorization.k8s.io
```

as you see 3 services get's created 
- Service Account
- Role
- Role Binding
to create the user apply the file into your kubernetes cluster 

```shell
kubectl apply -f viewer/rbac.yml
```
to generate token : 
```shell 
kubectl create token example-viewer
```

### 2- Manager User

to create a user with the manager role apply the example in manager/rbac.yml

```yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  namespace: default
  name: example-manager

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: example-manager-role
rules:
- apiGroups: [""]
  resources: ["pods", "namespaces"]
  verbs: ["get", "watch", "list"]
- apiGroups: ["chaos-mesh.org"]
  resources: [ "*" ]
  verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: example-manager-bind
  namespace: default
subjects:
- kind: ServiceAccount
  name: example-manager
  namespace: default
roleRef:
  kind: Role
  name: example-manager-role
  apiGroup: rbac.authorization.k8s.io
```
to create the user apply the file into your kubernetes cluster

```shell
kubectl apply -f manager/rbac.yml
```
to generate token :
```shell 
kubectl create token example-manager
```

## Token Duration 

Default token has a duration of 1 hour then it expires 
but we can configure the token duration while creation : 

```shell
kubectl create token example-manager --duration=4380h //example of 6 months 
```
## User permession revoke 

to revoke the token and make it inactive you'll have to delete the permession or the role binding if you want to keep the user 

- Delete the user with it's permession : 

```shell
kubectl delete -f rbac.yml
```

## recommendation

i recommend creating rbac files with the usernames so you can seperate each user and thier permession 

example : 
for example you have users named : 
```
bill
jordan
dave
```
you'll name their rbac files 
```
bill-rbac.yml
jordan-rbac.yml
dave-rbac.yml
```
and if you want to delte someone let's say dave you can just : 

```shell
kubectl delete -f dave-rbac.yml
```

and also instead of 'example' you can change in the yaml file to the user name 
example 

```yaml
kind: ServiceAccount
...
  name: dave-manager
---
kind: Role
...
  name: dave-manager-role
...
---
apiVersion: rbac.authorization.k8s.io/v1
..
  name: dave-manager-bind
.....
```
