# JHipster-generated Kubernetes configuration

## Preparation
## my Jang Woong for docker registry

You will need to push your image to a registry. If you have not done so, use the following commands to tag and push the images:

```
$ docker image tag book gcr.io/cnaps-project/book
$ docker push gcr.io/cnaps-project/book
$ docker image tag bookcatalog gcr.io/cnaps-project/bookcatalog
$ docker push gcr.io/cnaps-project/bookcatalog
$ docker image tag gateway gcr.io/cnaps-project/gateway
$ docker push gcr.io/cnaps-project/gateway
$ docker image tag rental gcr.io/cnaps-project/rental
$ docker push gcr.io/cnaps-project/rental
```

```
$ cd gateway
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=192.168.1.10:8443/gateway:latest
$ cd book
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=192.168.1.10:8443/book:latest
$ cd bookCatalog
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=192.168.1.10:8443/bookcatalog:latest
$ cd rental
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=192.168.1.10:8443/rental:latest
$ cd board
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=192.168.1.10:8443/board:latest

docker images 192.168.1.10:8443/book
docker push 192.168.1.10:8443/book

docker images 192.168.1.10:8443/bookcatalog
docker push 192.168.1.10:8443/bookcatalog

docker images 192.168.1.10:8443/rental
docker push 192.168.1.10:8443/rental

docker images 192.168.1.10:8443/gateway
docker push 192.168.1.10:8443/gateway


cd book
sed -i 's/"image: gcr.io/cnaps-project"/"image: 192.168.1.10:8443/book"/' book-deployment.yml
```
## Deployment

쿠버네티스용 Continuous Deployment 툴인 Skaffold
 
You can deploy all your apps by running the below bash command:

```
./kubectl-apply.sh -f (default option)  [or] ./kubectl-apply.sh -k (kustomize option) [or] ./kubectl-apply.sh -s (skaffold run)
```

If you want to apply kustomize manifest directly using kubectl, then run

```
kubectl apply -k ./
```

If you want to deploy using skaffold binary, then run

```
skaffold run [or] skaffold deploy
```

## Exploring your services

Use these commands to find your application's IP addresses:

```
$ kubectl get svc gateway
```

## Scaling your deployments

You can scale your apps using

```
$ kubectl scale deployment <app-name> --replicas <replica-count>
```

## zero-downtime deployments

The default way to update a running app in kubernetes, is to deploy a new image tag to your docker registry and then deploy it using

```
$ kubectl set image deployment/<app-name>-app <app-name>=<new-image>
```

Using livenessProbes and readinessProbe allow you to tell Kubernetes about the state of your applications, in order to ensure availablity of your services. You will need minimum 2 replicas for every application deployment if you want to have zero-downtime deployed.
This is because the rolling upgrade strategy first stops a running replica in order to place a new. Running only one replica, will cause a short downtime during upgrades.

## JHipster registry

The registry is deployed using a headless service in kubernetes, so the primary service has no IP address, and cannot get a node port. You can create a secondary service for any type, using:

```
$ kubectl expose service jhipster-registry --type=NodePort --name=exposed-registry
```

and explore the details using

```
$ kubectl get svc exposed-registry
```

For scaling the JHipster registry, use

```
$ kubectl scale statefulset jhipster-registry --replicas 3
```

## Troubleshooting

> my apps doesn't get pulled, because of 'imagePullBackof'

Check the docker registry your Kubernetes cluster is accessing. If you are using a private registry, you should add it to your namespace by `kubectl create secret docker-registry` (check the [docs](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) for more info)

> my applications are stopped, before they can boot up

This can occur if your cluster has low resource (e.g. Minikube). Increase the `initialDelaySeconds` value of livenessProbe of your deployments

> my applications are starting very slow, despite I have a cluster with many resources

The default setting are optimized for middle-scale clusters. You are free to increase the JAVA_OPTS environment variable, and resource requests and limits to improve the performance. Be careful!
