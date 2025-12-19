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

5. 