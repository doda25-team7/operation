## How to deploy everyting into the cluster using the helm chart

1. Open the ctrl VM terminal shell
```bash
vagrant ssh ctrl
```

2. Change to operation directory
```bash
cd operation
```

3. Run the Helm install command
```bash
helm upgrade --install app ./SMS-checker -n sms --create-namespace
```


## How to enable monitoring using prometheus

1. Open the ctrl VM terminal shell
```bash
vagrant ssh ctrl
```

2. Add prometheus to the helm repo
```bash
helm repo add prom-repo https://prometheus-community.github.io/helm-charts
```

3. Update the helm repo
```bash
helm repo update
```

4. Install prometheus into a dedicated namespace
```bash
helm install myprom prom-repo/kube-prometheus-stack \
  -n monitoring \
  --create-namespace
```

5. Deploy the application (make sure the your not in the /vagrant/operation directory inside the VM after this step)
```bash
cd /vagrant/operation

helm upgrade --install app ./SMS-checker -n sms --create-namespace
```

6. To access Prometheus UI
```bash
kubectl port-forward -n monitoring svmyprom-kube-prometheus-sta-prometheus   --address 0.0.0.0  us 9090:9090
```

7. To get /metrics (have two terminals open, one for the port forward, one for running the curl command)
```bash
kubectl port-forward -n sms svc/app-app-service 8080:8080

curl http://localhost:8080/sms/metrics
```

8. To get /metrics in browser
In one terminal do:
```bash
vagrant ssh ctrl -- -L 8080:localhost:8080
```
In a second terminal do:
```bash
kubectl port-forward -n sms svc/app-app-service 8080:8080
```
Open http://localhost:8080/sms/metrics in browser.