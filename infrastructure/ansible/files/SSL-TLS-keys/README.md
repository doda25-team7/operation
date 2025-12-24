# Dummy SSL/TLS keys

This folder contains the dummy SSL/TLS keys that are deployed into the HTTPS Ingress Controller. These keys are located here instead of generated on the fly soly to meet the requirements of the assignement given. 

In actual production setting, SSL/TLS keys should be deployed using an ACME client and server. 

The keys where generated using:
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout doda.local.key -out doda.local.crt -subj "/CN=doda.local/O=doda.local" -addext "subjectAltName = DNS:doda.local"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout dashboard.local.key -out dashboard.local.crt -subj "/CN=dashboard.local/O=dashboard.local" -addext "subjectAltName = DNS:dashboard.local"
```