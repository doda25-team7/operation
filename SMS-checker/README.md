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
kubectl port-forward -n monitoring svc/myprom-kube-prometheus-sta-prometheus \
  --address 0.0.0.0 \
  9090:9090
```
Then open http://192.168.56.100:9090

7. To get /metrics (have two terminals open, one for the port forward, one for running the curl command)
```bash
kubectl port-forward -n sms svc/app-app-service 8080:8080

curl http://localhost:8080/metrics
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
Open http://localhost:8080/metrics in browser.


### How to enable alerting to Discord.

1. Enabling alerting by updating Prometheus installation.
```bash
kubectl label namespace sms alertmanager-config=enabled --overwrite
helm upgrade myprom prom-repo/kube-prometheus-stack \
  -n monitoring \
  --set alertmanager.alertmanagerSpec.alertmanagerConfigNamespaceSelector.matchLabels.alertmanager-config=enabled
```

2. Create webhook secret.
```bash
kubectl create secret generic alertmanager-discord-webhook \
  --from-literal=webhook-url='YOUR_DISCORD_WEBHOOK_URL' \
  -n sms
```

3. Deploy the application
```bash
helm upgrade --install app ./SMS-checker -n sms --create-namespace
```

## Traffic Management (Istio)

### Testing Traffic Splitting (90/10 Canary)
```bash
# Run multiple requests - expect ~90% stable, ~10% canary
for i in {1..20}; do
  curl -s -H "Host: doda.local" http://192.168.56.91/ -D - 2>&1 | grep "x-version"
done
```

### Testing Sticky Sessions
```bash
# Get assigned a version
VERSION=$(curl -s -H "Host: doda.local" http://192.168.56.91/ -D - 2>&1 | grep -i "x-version" | awk '{print $2}' | tr -d '\r')
echo "Assigned: $VERSION"

# Subsequent requests stay on same version
for i in {1..5}; do
  curl -s -H "Host: doda.local" -H "x-version: $VERSION" http://192.168.56.91/ -D - 2>&1 | grep "x-version"
done
```

## Shadow Launch

Shadow launch mirrors traffic to a shadow model-service for testing new model versions without exposing to users.

### Verify Shadow Pod is Running
```bash
kubectl get pods -l version=shadow
```

### Check Shadow Receives Mirrored Traffic
```bash
# Make requests to the app
for i in {1..5}; do
  curl -s -H "Host: doda.local" http://192.168.56.91/
done

# Check shadow pod logs - should see the same requests
kubectl logs -l app=model-service,version=shadow --tail=20
```

### Compare Stable vs Shadow Logs
```bash
# Stable model logs
kubectl logs -l app=model-service,version=stable --tail=10

# Shadow model logs (should have same requests)
kubectl logs -l app=model-service,version=shadow --tail=10
```
