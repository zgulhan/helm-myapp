It's a CI/CD pipeline for dev, test, and prod environments built with GitLab, Helm and ArgoCD.

# CI Pipeline

For the CI part please clone this [project](https://github.com/zgulhan/rest-api) to your GitLab repository. 

In GitLab, after adding $REGISTRY_PASS which is your Docker Hub's password as a variable from Settings -> CI/CD -> Variables, automatic image building will be triggered.

![masked variable](/pictures/1.png)

# CD Pipeline

> For the CD part, I used **Minikube** on a Docker driver. Please set your environment through [instructions](https://minikube.sigs.k8s.io/docs/start/) before move on.

You need to set your credentials as a Kubernetes Secret to pull an image from private registry:

```
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1 --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>

```

> For more information about pulling an image from private registry, please visit the [link](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-in-the-cluster-that-holds-your-authorization-token).

## Helm Chart Logic

A custom developed helm chart is used for this project. It contains very basic attributes for a container like image, tag, port, configmap and secret.

Configmap is used to differentiate three environment from each other and change DB to avoid rebuilding the app's source code.

After pulling this repository to your local, please configure your image and tag name of your registry through:
- values.yaml
- /test/values-test.yaml
- /prod/values-prod.yaml

For my example, I used **zdgul/myapp:rest-api-4.5** image but since it is a private registry, you need to use your own registry to pull image.

After then, please install both ArgoCD CLI and UI through [instructions](https://faun.pub/argo-cd-helm-the-gitops-way-of-deploying-applications-af158420bde5).


# Dev Environment Setting

```
argocd app create myhelmapp --repo https://github.com/zgulhan/helm-myapp.git --path myhelmapp --dest-server https://kubernetes.default.svc --dest-namespace default

argocd app sync myhelmapp

kubectl port-forward svc/rest-api -n default 3000:3000
```

> Please change --repo with your own repository.

After commands above, in [localhost:3000](localhost:3000), you will see this:
![dev environment](/pictures/2.png)

# Test Environment Setting
```
argocd app create myhelmapptest --repo https://github.com/zgulhan/helm-myapp.git --path myhelmapp/test --values values-test.yaml --dest-server https://kubernetes.default.svc --dest-namespace test

argocd app sync myhelmapptest

kubectl port-forward svc/rest-api -n test 3000:3000
```

> Please change --repo with your own repository.

After commands above, in [localhost:3000](localhost:3000), you will see this:
![test environment](/pictures/3.png)

# Prod Environment Setting
```
argocd app create myhelmappprod --repo https://github.com/zgulhan/helm-myapp.git --path myhelmapp/prod --values values-prod.yaml --dest-server https://kubernetes.default.svc --dest-namespace prod

argocd app sync myhelmappprod

kubectl port-forward svc/rest-api -n prod 3000:3000

```

> Please change --repo with your own repository.

After commands above, in [localhost:3000](localhost:3000), you will see this:
![prod environment](/pictures/4.png)

# DB Change

In [localhost:3000/products](localhost:3000/products), you can access the API:

![products](/pictures/5.png)

If you want to change your DB without rebuilding app image, please change **configmap.data.DBURL** in **values.yaml** file.


# Conclusion

![argocd](/pictures/6.png)

You can check three deployments in ArgoCD UI like above. Sync policy is designed as manual to provide more control over the project. After any change in the helm repository, deployments are required to be synced in CLI or UI.

This is the end of the project. Thank you for your interest.

# Helpful Resources
- [Minikube Instructions](https://minikube.sigs.k8s.io/docs/start/)
- [Pulling an Image from Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-in-the-cluster-that-holds-your-authorization-token)
- [ArgoCD Instructions](https://faun.pub/argo-cd-helm-the-gitops-way-of-deploying-applications-af158420bde5)