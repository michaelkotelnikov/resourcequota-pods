# ResourceQuota and pods in action

This repository seeks to demonstrate how resource quotas in Kubernetes may affect the rolling strategies in Deployment resources.

### Setup

In order to run the demonstration, follow the next steps -

1. Create a namespace

```
$ kubectl create namespace quota-test
```

2. Create the ResourceQuota resource in the repository -

```
$ kubectl create -f resourcequota.yaml -n quota-test
```

### Recreate strategy

1. Create the deployment with the recreate strategy -

```
$ kubectl apply -f recreate-deployment/deployment.yaml -n quota-test
```

2. Verify the pod is running -

```
$ kubectl get pods -n quota-test

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7949948f5f-nfnhb   1/1     Running   0          2s
```

3. Modify `image: bitnami/nginx:1.25.1` to `image: bitnami/nginx:1.26.1` in the deployment instance.

4. Verify that the change has initiated a new pod with the new version.

```
$ kubectl get pods -n quota-test

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7b4d8787d8-8mgtt   1/1     Running   0          5s
```

5. Remove the deployment

```
$ kubectl delete -f recreate-deployment/deployment.yaml -n quota-test
```

### RollingUpdate strategy

1. Create the deployment with the rollingupdate strategy -

```
$ kubectl apply -f rolling-deployment/deployment.yaml -n quota-test
```

2. Verify the pod is running -

```
$ kubectl get pods -n quota-test

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7995bdfcd9-rq649   1/1     Running   0          5s
```

3. Modify `image: bitnami/nginx:1.25.1` to `image: bitnami/nginx:1.26.1` in the deployment instance.

4. Verify that the change has **not** initiated a new pod with the new version.

```
$ kubectl get pods -n quota-test

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7995bdfcd9-rq649   1/1     Running   0          25s
```

5. Check the events in the namespace, verify that the quota is blocking the rollout -

```
$ kubectl get events -n quota-test

...
18s         Warning   FailedCreate        replicaset/nginx-deployment-576898f6cd   Error creating: pods "nginx-deployment-576898f6cd-zvtm2" is forbidden: exceeded quota: quota, requested: limits.cpu=1,limits.memory=512Mi,requests.cpu=100m,requests.memory=128Mi, used: limits.cpu=1,limits.memory=512Mi,requests.cpu=100m,requests.memory=128Mi, limited: limits.cpu=1,limits.memory=512Mi,requests.cpu=100m,requests.memory=128Mi
...
```

6. Delete the created deployments -

```
$ kubectl delete -f rolling-deployment/deployment.yaml -n quota-test
```

### Conclusion

If a developer is using the RollingUpdate strategy it must have resources available in the ResourceQuota to fit the largest pod that is going to roll out.
