---
title: "Deploying an application"
sidebar_position: 15
---

We have successfully configured Argo CD on our cluster so now we can deploy an application. To demonstrate the difference between a GitOps-based delivery of an application and other methods, we'll migrate the UI component of the sample application which is currently using the `kubectl apply -k` approach to the new Argo CD deployment approach.

First let's remove the existing UI component so we can replace it:

```bash
$ kubectl delete -k /workspace/manifests/ui
namespace "ui" deleted
serviceaccount "ui" deleted
configmap "ui" deleted
service "ui" deleted
deployment.apps "ui" deleted
```

Next, clone the CodeCommit repository:

```bash
$ cd ~/environment
$ git clone ssh://${GITOPS_IAM_SSH_KEY_ID}@git-codecommit.${AWS_DEFAULT_REGION}.amazonaws.com/v1/repos/${EKS_CLUSTER_NAME}-gitops ~/environment/gitops
$ cd gitops
$ git checkout -b argocd
Switched to a new branch 'argocd'
$ git add . && git commit -m "Initial commit"
$ git push --set-upstream origin argocd
$ cd ..
```

Now, let's get into the cloned repository and start creating our GitOps configuration. Copy the existing kustomize configuration for the UI service:

```bash
$ mkdir ~/environment/gitops/apps
$ cp -R /workspace/manifests/ui ~/environment/gitops/apps
```

We'll then need to create a kustomization in the `apps` directory:

```file
automation/gitops/argocd-codecommit/kustomization.yaml
```

Copy this file to the Git repository directory:

```bash
$ cp /workspace/modules/automation/gitops/argocd-codecommit/kustomization.yaml ~/environment/gitops/apps/kustomization.yaml
```

You Git directory should now look something like this which you can validate by running `tree ~/environment/gitops`:

```
.
└── apps
    ├── kustomization.yaml
    └── ui
        ├── configMap.yaml
        ├── deployment.yaml
        ├── kustomization.yaml
        ├── namespace.yaml
        ├── serviceAccount.yaml
        └── service.yaml

2 directories, 7 files
```

Finally we can push our configuration to AWS CodeCommit:

```bash
$ (cd ~/environment/gitops && \
git add . && \
git commit -am "Adding the UI service" && \
git push)
```

It will take Argo CD some time to notice the changes in CodeCommit and reconcile. You can use the Argo CD UI to `Sync` for our new `apps` kustomization to appear.

Now, let's login to Argo CD UI with `admin`, explore and Sync changes:

Get Argo CD UI url and `admin` password

```bash
$ echo "ArgoCD URL: http://$(kubectl get svc argo-cd-argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname')"
$ echo "ArgoCD admin password: $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)"
```

That shows that Argo CD created the basic kustomization, and that it's in sync with the cluster.

We've now successfully migrated the UI component to deploy using Argo CD, and any further changes pushed to the Git repository will be automatically reconciled to our EKS cluster.

You should now have all the resources related to the UI services deployed once more. To verify, run the following commands:

```bash
$ kubectl get deployment -n ui ui
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     1/1     1            1           5m
$ kubectl get pod -n ui
NAME                  READY   STATUS    RESTARTS   AGE
ui-54ff78779b-qnrrc   1/1     Running   0          5m
```