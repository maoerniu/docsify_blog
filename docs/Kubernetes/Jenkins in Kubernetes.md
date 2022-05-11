# Jenkins in Kubernetes

`kubectl create namespace jenkins`

```
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
```

`helm search repo jenkinsci`


## Create a persistent volume
jenkins-pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: jenkins
spec:
  storageClassName: rook-ceph-block
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

`kubectl apply -f jenkins-pvc.yaml`

## Create a service account
jenkins-sa.yaml
https://raw.githubusercontent.com/installing-jenkins-on-kubernetes/jenkins-sa.yaml
`kubectl apply -f jenkins-sa.yaml`

## Install Jenkins
```
chart=jenkinsci/jenkins
helm install jenkins -n jenkins -f jenkins-values.yaml $chart
```