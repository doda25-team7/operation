# Vagrant Kubernetes Setup

This directory contains the Vagrant configuration for setting up a Kubernetes cluster with VirtualBox.

## Prerequisites

- [Vagrant](https://www.vagrantup.com/downloads) installed 
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed

```bash
sudo apt update
sudo apt install virtualbox
```

## Test installations
```bash
vagrant --version
vboxmanage --version
```

## Configuration

The setup creates the following VMs:

- **ctrl**: Control node (1 core, 4GB RAM) - Kubernetes controller
- **node-1, node-2, ...**: Worker nodes (2 cores, 6GB RAM each) - Kubernetes workers

### Scaling Workers

To change the number of worker nodes, edit the `NUM_WORKERS` variable in the `Vagrantfile`:

```ruby
NUM_WORKERS = 2  # Change this number to scale up or down
```

## Network Configuration

- Control node: `192.168.56.100`
- Worker node-N: `192.168.56.{100+N}`
- Ingress controller: `192.168.56.90`

## Usage

### Start all VMs
```bash
vagrant up
```

### Finalizing the Cluster Setup

```bash
ansible-playbook -i ansible/inventory.ini ansible/finalization.yaml
```
### If KVM is conflicting temporarily disable it 
```bash
sudo modprobe -r kvm_intel kvm
```

### Start specific VM
```bash
vagrant up ctrl
vagrant up node-1
```

### SSH into a VM
```bash
vagrant ssh ctrl
vagrant ssh node-1
```

### Check VM status
```bash
vagrant status
```

### Stop VMs
```bash
vagrant halt
```

### Destroy VMs
```bash
vagrant destroy -f
```

### Reload VMs (after config changes)
```bash
vagrant reload
```

## Resource Adjustments

Resources and VM configs can be adjusted in the `Vagrantfile`:

- For control node: Modify `vb.memory` and `vb.cpus` in the `ctrl` block
- For worker nodes: Modify `vb.memory` and `vb.cpus` in the worker node block

## Check the network status
log in the ctrl node and do `kubectl get nodes`

## Access the Dashboard

### Step 1: Add dashboard.local to your hosts file

**On macOS/Linux:**
```bash
sudo nano /etc/hosts
```
Add this line at the end of the file:
```
192.168.56.90    dashboard.local
```
Save and exit (Ctrl+X, then Y, then Enter in nano).

**On Windows:**
1. Open Notepad as Administrator
2. Open `C:\Windows\System32\drivers\etc\hosts`
3. Add this line at the end:
```
192.168.56.90    dashboard.local
```
4. Save the file

### Step 2: Verify the cluster is ready

First, make sure your VMs are running and the finalization playbook has been executed:
```bash
vagrant status
```

If the finalization hasn't been run yet, execute it:
```bash
cd infrastructure
ansible-playbook -i ansible/inventory.ini ansible/finalization.yaml
```

### Step 3: Get the authentication token

SSH into the control node:
```bash
vagrant ssh ctrl
```

Once inside the ctrl VM, generate the token:
```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Copy the entire token that is displayed (it will be a long string).

### Step 4: Access the dashboard

1. Open your web browser
2. Navigate to: `http://dashboard.local/`
3. You should see a login page asking for a token
4. Paste the token you copied in Step 3
5. Click "Sign in"

### Troubleshooting

**If `http://dashboard.local/` doesn't work:**

1. First, verify the hosts entry:
   ```bash
   # On macOS/Linux
   cat /etc/hosts | grep dashboard.local
   
   # Should show: 192.168.56.90    dashboard.local
   ```

2. Try accessing by IP directly:
   ```bash
   # Test if the IP is reachable
   ping 192.168.56.90
   ```

3. If ping fails, check if the ingress controller is running:
   ```bash
   vagrant ssh ctrl
   kubectl get pods -n ingress-nginx
   kubectl get svc -n ingress-nginx
   ```

4. If needed, manually add the IP to the control node's interface:
   ```bash
   vagrant ssh ctrl
   sudo ip addr add 192.168.56.90/24 dev eth1
   ```

5. Verify the dashboard ingress is configured:
   ```bash
   vagrant ssh ctrl
   kubectl get ingress -n kubernetes-dashboard
   ```

**If you get "Unauthorized (401): Invalid credentials provided" error:**

1. First, verify the ingress is pointing to the correct service:
   ```bash
   vagrant ssh ctrl
   kubectl get ingress -n kubernetes-dashboard -o yaml
   kubectl get svc -n kubernetes-dashboard
   ```
   The ingress should point to service `kubernetes-dashboard-kong-proxy` on port 443.

2. Regenerate a fresh token (tokens expire quickly):
   ```bash
   vagrant ssh ctrl
   kubectl -n kubernetes-dashboard create token admin-user
   ```
   Copy the entire token immediately and use it within a few minutes.

3. Verify the admin-user ServiceAccount exists and has proper permissions:
   ```bash
   vagrant ssh ctrl
   kubectl get sa -n kubernetes-dashboard admin-user
   kubectl get clusterrolebinding admin-user
   ```

4. If the issue persists, try accessing the dashboard directly via port-forward (bypassing ingress):
   ```bash
   vagrant ssh ctrl
   kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8443:443
   ```
   Then access `https://localhost:8443` in your browser (accept the self-signed certificate warning) and try the token again.

5. If you've updated the finalization.yaml file, re-run the finalization playbook:
   ```bash
   cd infrastructure
   ansible-playbook -i ansible/inventory.ini ansible/finalization.yaml
   ```

**If you get "503 Service Temporarily Unavailable" error:**

1. Check if the dashboard service exists and has endpoints:
   ```bash
   vagrant ssh ctrl
   kubectl get svc -n kubernetes-dashboard
   kubectl get endpoints -n kubernetes-dashboard kubernetes-dashboard-kong-proxy
   kubectl get pods -n kubernetes-dashboard
   ```
   The service `kubernetes-dashboard-kong-proxy` should have endpoints (IP addresses), and pods should be in "Running" state.

2. Verify the ingress is pointing to the correct service and port:
   ```bash
   vagrant ssh ctrl
   kubectl get ingress -n kubernetes-dashboard -o yaml
   kubectl get svc -n kubernetes-dashboard kubernetes-dashboard-kong-proxy -o yaml
   ```
   Check that:
   - The ingress service name is `kubernetes-dashboard-kong-proxy`
   - The ingress port matches the service port (should be 443)

3. Check ingress controller logs for errors:
   ```bash
   vagrant ssh ctrl
   kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller --tail=50
   ```
   Look for connection errors or backend errors.

4. Test direct access to the dashboard service (bypassing ingress):
   ```bash
   vagrant ssh ctrl
   kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard-kong-proxy 8443:443
   ```
   Then try accessing `https://localhost:8443` in your browser (accept the self-signed certificate warning). If this works, the issue is with the ingress configuration.

5. If the service port is 8443 instead of 443, you may need to update the ingress manually:
   ```bash
   vagrant ssh ctrl
   kubectl edit ingress -n kubernetes-dashboard kubernetes-dashboard
   ```
   Change the port number from 443 to 8443 if needed.

6. Re-run the finalization playbook to ensure everything is configured correctly:
   ```bash
   cd infrastructure
   ansible-playbook -i ansible/inventory.ini ansible/finalization.yaml
   ```

**If vagrant up gets stuck on the task "Run Join Command" for node-1**

1. Destroy VMs
   ```bash
   vagrant destroy -f
   ```

2. Open VirtualBox GUI
3. Go to network tab
4. Under "Host-only Networks" remove the network corresponding to the destroyed VMs (Usually it's vboxnet0)

5. Start up VMs
   ```bash
   vagrant up
   ```

**If the IP adress configured for configured for the host-only network is not within the allowed ranges**

1. Destroy VMs
   ```bash
   vagrant destroy -f
   ```

2. Open the VirtualBox host-only network configuration file
   ```bash
   sudo nano /etc/vbox/networks.conf
   ```

3. Add following lines into the file:
      * 10.0.0.0/8 192.168.0.0/16
      * 2001::/64

4. Start up VMs
   ```bash
   vagrant up
   ```

## NOTES
login the ctrl and use `systemctl restart` to restart failed service of any service failed in k8s

# Kubernetes Setup

## Kubernetes vs Docker Compose

| Docker Compose | Kubernetes |
|----------------|------------|
| Single machine | Multiple machines |
| Manual restart | Auto-restart |
| No load balancing | Built-in |
| Port mapping | Ingress routing |

Docker is good for small projects that run on one machine, but when more complexity, load balancing and failure recovery is needed kubernetes is the way to go.

## What Each Resource Does

### Deployment
Runs and manages your containers (Pods). If a Pod crashes, Deployment automatically restarts it. It's like a supervisor that keeps your the app running.

### Service
Gives the Pods a stable DNS name. Pods have changing IPs, but Service provides a permanent address like `http://model-service:8081`.

### Ingress
Routes external traffic into the cluster. Maps `http://doda.local` → `app-service`.

### ConfigMap
Stores configuration (non-sensitive data) like URLs, ports, feature flags. One ConfigMap can be used by multiple Pods.

### Secrets 
TODO

## How They Connect

The HTTP(S) requests get passed in the following order:
```
User → Ingress (ingress-nginx) → Service (app-service) → Deployment → Pod (container)
```
The ingress-nginx service is alloced its IP adres by the metallb-controller, and the routing is configured over ARP (protocal "layer2") by the metallb-speaker. 

## Quick Commands

```bash
# Deploy everything using kubectl
kubectl apply -f kubernetes-basic.yaml

# Deploy everything using Helm
helm upgrade app SMS-checker

# Check status
kubectl get deployments,services,ingress,pods

# View logs
kubectl logs -l app=app-service

# Scale up
kubectl scale deployment app-service --replicas=3
```