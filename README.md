
# 📦 Deploying Gitea and Runners on Kubernetes

This repository contains configuration files to install and configure a [Gitea](https://gitea.io) instance and its runners on Kubernetes using Helm.

---


## Gitea (Self-hosted) vs GitHub Team – Cost and Feature Comparison

Setup for comparasion:

| Item | Specification
| ------------|------------------------------|
| Main Server | 4 vCPUs, 8 GB RAM, 100 GB disk (supports well 100 users and 30 repositories)|
| Runner      | 2 vCPUs, 4 GB RAM, 100 GB disk                                              |


Estimatives:


| Item                    | **Gitea (Self-hosted)**                                                   | **GitHub Team**                            |
| ----------------------- | ------------------------------------------------------------------------- | ------------------------------------------ |
| 💵 **License**          | Free                                                                      | US\$ 400/month (100 users)                 |
| ⚙️ **CI/CD**            | \~US\$ 20 (extra VM/container)                                            | \~US\$ 4 (GitHub Actions extra minutes)    |
| 🖥️ **Infrastructure**  | \~US\$ 55 (VM, disk, backup)                                              | Included                                   |
| 💰 **Total (estimate)** | **US\$ 175–375/month**                                                    | **US\$ 404/month**                         |
| 🔧 **Maintenance**      | Fully managed by your team                                                | Managed by GitHub                          |
| 🛠️ **CI/CD options**   | Gitea Actions (beta) or external tools (Jenkins, Woodpecker, Drone, etc.) | GitHub Actions (native)                    |
| 🔐 **Privacy**          | Full control (you host the code and data)                                 | Data hosted by GitHub/Microsoft (US-based) |
| 📈 **Scalability**      | You manage infra, tuning, scaling                                         | Automatically managed by GitHub            |
| 🧠 **Learning curve**   | Higher (setup, CI integration, updates)                                   | Lower (plug-and-play experience)           |
          


You can find here a detailed comparation and [comparated to other Git Hosting](https://docs.gitea.com/installation/comparison)


### Conclusion

Gitea is more affordable, even when considering CI/CD and infrastructure costs.
GitHub is more convenient, offering native integration, support, zero maintenance, and good scalability.

**If you already have a team or prefer more control over your data and want to save money, Gitea is definitely worth it.
If your team values speed, integration, and less concern with infrastructure, GitHub is worth the extra cost.**



## 📁 Repository Structure

```
.
├── gitea/
│   ├── values.yaml       # Custom configuration for Gitea via Helm
│   └── Chart.yaml        # Basic chart metadata
├── runner/
│   ├── values.yaml       # Configuration for the Actions runner
│   
└── README.md             # This file
│   
└── .gitignore             # Git config file
```

---

## 🚀 Prerequisites

- A running Kubernetes cluster
- `kubectl` configured
- [Helm](https://helm.sh) installed
- Gitea Helm repository added:

```bash
helm repo add gitea-charts https://dl.gitea.io/charts/
helm repo update
```

---

## 🛠️ Deploying Gitea

1. Go to the `gitea` directory:

```bash
cd gitea
```

2. Install Gitea with Helm:

```bash
helm install gitea gitea-charts/gitea -f values.yaml -n gitea
```

3. Wait for the pods to be ready:

```bash
kubectl get pods -n gitea
```

4. (Optional) Access Gitea using port-forward:

```bash
kubectl port-forward svc/gitea-http 3000:3000 -n gitea
```

Then open [http://localhost:3000](http://localhost:3000) in your browser.

---

## ⚙️ Configuring `ROOT_URL`

After installation, ensure the `ROOT_URL` value in Gitea's `app.ini` matches the internal address:

Example:

```
ROOT_URL = http://gitea-http.default.svc.cluster.local:3000/
```

---

## 🤖 Deploying the Runner

1. Go to the `runner` directory:

```bash
cd runner
```

2. Get a runner registration token, you can do it with API request or via gitea UI:

```bash
curl -X POST   -H "Authorization: token <YOUR_ACCESS_TOKEN>"   http://gitea-http.default.svc.cluster.local:3000/api/v1/actions/runners/registration-token
```

3. Insert this token into the runner's `values.yaml` file:

```yaml
env:
  - name: GITEA_INSTANCE_URL
    value: "http://gitea-http.default.svc.cluster.local:3000"
  - name: REGISTRATION_TOKEN
    value: "<your token here>"
```

4. Install the runner with Helm:

```bash
helm install gitea-runner gitea-charts/gitea-runner -f values.yaml -n gitea
```

---

## ✅ Verifying

- Log into Gitea
- Go to **Settings > Runners** to check if the runner is registered

---

## 🧹 Cleanup

```bash
helm uninstall gitea -n gitea
helm uninstall gitea-runner -n gitea
```

---

## 📚 References

- [Gitea Official Documentation](https://docs.gitea.io/)
- [Gitea Helm Charts](https://gitea.com/gitea/helm-chart)
- [Gitea Compared to other Git hosting](https://docs.gitea.com/installation/comparison)

