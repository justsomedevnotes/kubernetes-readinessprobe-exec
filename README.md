# Introduction
This post demonstrates an exec readinessProbe.  It uses a check for DNS to establish readiness.  Other kinds of exec probes could be the existance of a file, access to a database, etc.  A Kubernetes NetworkPolicy is used to demonstrate the probes functionality.  A CNI that supports Kubernetes NetworkPolicy such as NSX-T, Calico, Weave is required for the NetworkPolicy test.  Another test could be to just use an invalid URL.

## Create the Pod and ReadinessProbe
This pod is just a simple busybox image with a exec readinessProbe that performs an nslookup on google.com.  If the nslookup fails the readinessProbe fails and flags the pod as not ready.

```console
kind: Pod
apiVersion: v1
metadata:
  name: my-pod
  namespace: default
spec:
  containers:
  - name: c1
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
    - sleep
    - '3600'
    readinessProbe:
      periodSeconds: 5
      exec:
        command:
        - '/bin/sh'
        - '-c'
        - 'nslookup www.google.com'

```
Now create the pod.
```console
kubectl apply -f my-pod.yml
pod/my-pod created
```
Now check the status.  Notice the READY value is 1/1.
```console
kubectl get po
NAME      READY     STATUS    RESTARTS   AGE
my-pod    1/1       Running   0          53s
```

## Verify the Probe's Functionality
This test creates a NetworkPolicy that blocks all pods egress traffic from the default namespace.  After creating the policy notice the READY status of the pod changes from 1/1 to 0/1.

```console
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: my-policy
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  

```
Create the NetworkPolicy
```console
kubectl create -f my-netp.yml
networkpolicy.networking.k8s.io/my-policy created
```
Wait a few seconds and then check the pod READY status.
```console
kubectl get po
NAME      READY     STATUS    RESTARTS   AGE
my-pod    0/1       Running   0          1m
```


## Conclusion
This is a pretty simple demonstration of using readinessProbes to verify if a Pod is ready for use.  Typically readinessProbes are used in conjunction with a Kubernetes Service.

