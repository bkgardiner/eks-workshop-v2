---
title: "Install JupyterHub"
sidebar_position: 30
description: "In this section we will install JupyterHub"
---

```bash
$ helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
$ helm repo update

"jupyterhub" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jupyterhub" chart repository
Update Complete. *Happy Helming!*
```

```bash
$ helm upgrade --cleanup-on-fail \
  --install jupyterhub jupyterhub/jupyterhub \
  --namespace jupyterhub \
  --create-namespace \
  --version=3.3.8 \
  --values ~/environment/eks-workshop/modules/aiml/makemore/jupyterhub/config.yaml
```
