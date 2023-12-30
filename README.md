```
kubectl create namespace test

kubectl create namespace prod

helm install -f values.yaml restapi ./

helm install -f values-test.yaml restapitest ./ -n test

helm install -f values-prod.yaml restapiprod ./ -n prod

helm ls --all-namespaces
```
