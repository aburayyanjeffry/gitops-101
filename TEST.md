## Install
If you need to test the chart **locally** 
bash```
kubectl create namespace nginx-test
helm install my-nginx-test ./charts/nginx -n nginx-test
kubectl get pods -n nginx-test
helm list -A
kubectl port-forward svc/my-nginx-test 8080:80 -n nginx-test
```

## Remove
Clean up the test
bash```
helm uninstall my-nginx-test -n nginx-test
kubectl delete namespace nginx-test
```
