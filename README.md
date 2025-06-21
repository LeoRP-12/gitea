
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
          


You can find here a detailed comparasion and [comparated to other Git Hosting](https://docs.gitea.com/installation/comparison)


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
│   ├── deployment.yaml   # Configuration for the Actions runner
│   
└── README.md             # This file
│   
└── .gitignore             # Git config file
└── docker-compose.yaml    # Docker compose file to install with docker
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

create a namespace to Gitea for a better organization:

```bash
kubectl create namespace gitea
````

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
ROOT_URL = http://gitea-http.gitea.svc.cluster.local:3000/
```

---

## 🤖 Deploying the Runner

1. Go to the `runner` directory:

```bash
cd runner
```

2. Get a runner registration token, you can do it with API request or via gitea UI:

```bash
curl -X POST   -H "Authorization: token <YOUR_ADMIN_ACCESS_TOKEN>"   http://localhost:3000/api/v1/admin/actions/runners/registration-token
```

3. Insert this token into the runner's `deployment.yaml` file:

```yaml
env:
  - name: GITEA_INSTANCE_URL
    value: "http://gitea-http.gitea.svc.cluster.local:3000"
  - name: REGISTRATION_TOKEN
    value: "<your token here>"
```

4. Install the runner with kubectl:

```bash
cd runner
kubectl apply -f deployment.yaml
```

---

## ✅ Verifying

- Log into Gitea
- Go to **Settings > Runners** to check if the runner is registered

---

## 🧹 Cleanup

```bash
kubectl delete deployment gitea-runner -n gitea
helm uninstall gitea -n gitea
```

---

## Running with Docker


You can install your Gitea instance via Docker.

To launch your instance: 

```bash
docker-compose up -d
```

Check the URL http://localhost:3000 anf follow the instruction to finish the setup.


Create a repository and check if everthing is ok, if so the next step is configure your gitea runner.

Pull Gitea Runner Image:

```bash
docker pull docker.io/gitea/act_runner:latest # for the latest stable release
```


Get your runner registration token on **Settings > Actions > Runner** and run the command above to install gitea runner:

```bash
docker run \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e GITEA_INSTANCE_URL=http://host.docker.internal:3000/\
    -e GITEA_RUNNER_REGISTRATION_TOKEN=<Your registration token> \
    -e GITEA_RUNNER_NAME=gitea-runner \
    -e GITEA_RUNNER_LABELS=linux \
    --name gitea-runner \
    -d docker.io/gitea/act_runner:latest
```


You can test your Action creating a workflow file on path **.gitea/workflows/** you can check an example below:

```yaml
name: Gitea Actions Demo
run-name: ${{ gitea.actor }} is testing out Gitea Actions 🚀
on: [push]

jobs:
  Explore-Gitea-Actions:
    runs-on: linux
    container:
      image: node:20
    steps:
      - run: echo "🎉 The job was automatically triggered by a ${{ gitea.event_name }} event congrats."
      - run: echo "🐧 This job is now running on a ${{ runner.os }} server hosted by Gitea!"
      - run: echo "🔎 The name of your branch is ${{ gitea.ref }} and your repository is ${{ gitea.repository }}."
      - name: Check out repository code
        uses: actions/checkout@v4
      - run: echo "💡 The ${{ gitea.repository }} repository has been cloned to the runner."
      - run: echo "🖥️ The workflow is now ready to test your code on the runner."
      - name: List files in the repository
        run: |
          ls ${{ gitea.workspace }}
      - run: echo "🍏 This job's status is ${{ job.status }}."
```

## 🧹 Cleanup

```bash
docker stop gitea-runner
docker rm gitea-runner
docker-compose down
```



## 📚 References

- [Gitea Official Documentation](https://docs.gitea.io/)
- [Gitea Helm Charts](https://gitea.com/gitea/helm-chart)
- [Gitea Compared to other Git hosting](https://docs.gitea.com/installation/comparison)

