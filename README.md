# Monitoring, Logging, and Alerting on Kubernetes with Prometheus & Grafana

This repository deploys a complete **monitoring and alerting stack** on Amazon EKS using:

- Amazon EKS
- Prometheus (kube-prometheus-stack)
- Grafana
- Slack notifications via Alertmanager

It includes setup steps, Kubernetes configuration files, Slack alert integration, and a test alert to verify notifications.

---

## Prerequisites

- Linux/Ubuntu environment with `sudo` privileges
- AWS credentials configured
- Basic Kubernetes and Helm knowledge

---

## Install Required Tools

### Install AWS CLI
```bash
sudo snap install aws-cli --classic
```

### Install kubectl
```bash
sudo snap install kubectl --classic
```

### Install eksctl
```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

### Install Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## Create EKS Cluster
```bash
eksctl create cluster --name my-cluster --region us-east-1
```

---

## Deploy kube-prometheus-stack (Prometheus + Grafana)
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kps prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

Check pods and services:
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

---

## Expose Grafana & Prometheus via NodePort

### Patch Grafana
```bash
kubectl patch svc kps-grafana -n monitoring   -p '{"spec": {"type": "NodePort"}}'
```

### Patch Prometheus
```bash
kubectl patch svc kps-kube-prometheus-stack-prometheus -n monitoring   -p '{"spec": {"type": "NodePort"}}'
```

### Get Node IP
```bash
kubectl get nodes -o wide
```

---

## Grafana Access
```
URL: http://<NODE-IP>:<GRAFANA-PORT>
username: admin
password: $(kubectl get secret -n monitoring kps-grafana -o jsonpath="{.data.admin-password}" | base64 --decode)
```

---

## Clone the Repo
```bash
git clone https://github.com/sam-osung/Monitoring-Logging-and-Alerting-on-Kubernetes-with-Prometheus-and-Grafana.git
cd Monitoring-Logging-and-Alerting-on-Kubernetes-with-Prometheus-and-Grafana
```

## Configure Slack Alerts

```bash
kubectl apply -f slack-secret.yaml
kubectl apply -f alertmanager-slack.yaml
kubectl apply -f test-alert.yaml
```
✅ You should now receive a Slack alert notification.
---

## Troubleshooting

| Issue | Resolution |
|-------|------------|
| No Slack alerts | Verify webhook URL, check Alertmanager logs |
| Prometheus/Grafana unreachable | Check NodePort and external Node IP |
| Alerts not firing | Validate PrometheusRule status |

Check Alertmanager logs:
```bash
kubectl -n monitoring logs deploy/kps-kube-prometheus-stack-alertmanager
```

---

## Included Files

| File | Description |
|------|-------------|
| `slack-secret.yaml` | Slack webhook secret |
| `alertmanager-slack.yaml` | Alert routing config |
| `test-alert.yaml` | Rule to fire immediate test alert |

⚠️ Do not commit real secrets to public repositories.

---

## Contributing

Contributions are welcome — submit a PR or open an issue for improvements.
