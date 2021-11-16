# RBAC

In this exercise, you will create a configmap and try to get the config map from a pod, using a service account.


## Create a pod using a service account
```sh
kubectl create serviceaccount myapp
```

Create a pod associated with the service account `myapp`:
```sh
cat << EOF > pod-sa.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sa
spec:
  serviceAccountName: myapp-to-rule-the-world
  containers:
   - image: roffe/kubectl
     imagePullPolicy: Always
     name: kubectl
     command: [ "bash", "-c", "--" ]
     args: [ "while true; do sleep 30; done;" ]
EOF
kubectl create -f pod-sa.yaml
```

## Create a configmap and try to access it from the pod

Create a config map:
```sh
kubectl create configmap myconfig --from-literal data_1=foo
```

Get inside the pod and execute:
```sh
kubectl exec -it pod-sa -- bash
# then display the configmap
kubectl get configmap myconfig
```

Can you explain what happened ?

## Create a Role and a RoleBinding for the service account

Create a Role:
```sh
cat << EOF > role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: configmap-reader
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "watch", "list"]
EOF

kubectl create -f role.yaml
```

Create a RoleBinding:
```sh
cat << EOF > binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: configmap-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: configmap-reader
subjects:
- apiGroup: ""
  kind: ServiceAccount
  name: myapp
EOF

kubectl create -f binding.yaml
```

Try again to display the configMap inside the pod

Then try to change the configmap

Can you explain what happened ?

## Clean

```sh
kubectl delete -f pod-sa.yaml
kubectl delete -f binding.yaml
kubectl delete -f role.yaml
kubectl delete cm/myconfig
kubectl delete sa/myapp
```