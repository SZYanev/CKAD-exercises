![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)
# Observability (18%)

## Liveness and readiness probes

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)

### Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # our probe
      exec: # add this line
        command: # command definition
        - ls # ls command
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness # run this to see that liveness probe works
kubectl delete -f pod.yaml
```

</p>
</details>


### Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
      - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
    readinessProbe: # declare the readiness probe
      httpGet: # add this line
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness # to see the pod readiness details
kubectl delete -f pod.yaml
```

</p>
</details>

### Lots of pods are running in `qa`,`alan`,`test`,`production` namespaces.  All of these pods are configured with liveness probe.  Please list all pods whose liveness probe are failed in the format of `<namespace>/<pod name>` per line.

<details><summary>show</summary>
<p>

A typical liveness probe failure event
```
LAST SEEN   TYPE      REASON      OBJECT              MESSAGE
22m         Warning   Unhealthy   pod/liveness-exec   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

collect failed pods namespace by namespace

```sh  
kubectl get ns # check namespaces
kubectl -n qa get events | grep -i "Liveness probe failed"
kubectl -n alan get events | grep -i "Liveness probe failed"
kubectl -n test get events | grep -i "Liveness probe failed"
kubectl -n production get events | grep -i "Liveness probe failed"
```

</p>
</details>


## Debugging

### Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# show that there's an error
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>
