# Gitlab CI/CD pipeline on GKE example

Example of CI/CD within Gitlab presented on CodeCon 2019.

In this example we will create CI/CD pipeline on gitlab, deploying into Kubernetes cluster running on Google Cloud Platform. For That we need to prepare The infrastructure. Apart from Kubernetes cluster itself, we will use additional tools that will help us with automation of the whole process.

## Google Cloud Platform

As mentioned, this example is using Kubernetes cluster managed by Google Kubernetes Engine (GKE). In .gitlab-ci configuration file we are then using id of our GCP project as well as the name of our Kubernetes cluster. Furthermore we will use Google Container Registry to store our docker images. No configuration is needed to use GCR, but we need to create service account in IAM, give him sufficient permissions (at least for using GCR and deploying to GKE), and generate json credentials. This json will be used in gitlab secrets.

## Kubernetes cluster

In the cluster we will use namespaces for each environment and feature branches. Apart from that we will use few additional tools to further expand our automation. 

### Helm

Helm is a package manager for Kubernetes. We will use Helm firstly for deploy the following infrastructure tools and secondly to deploy our own application. We prepared templates for Kubernetes resources, creating our own Helm chart. Then we will just substitute few different parameters in our pipeline so we don't have to repeat all of the resources for each environment.

### Kube-lego

Tool that automatically requests Let's Encrypt certificates for Kubernetes ingresses. We will use this together with dns-external to automatically create secured dns records for newly deployed features.

[github.com/helm/charts/tree/master/stable/kube-lego](https://github.com/helm/charts/tree/master/stable/kube-lego)

Installed with (fill in different email):

```
helm upgrade --install  kube-lego stable/kube-lego --set rbac.create=true,config.LEGO_EMAIL=user@example.com,config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory
```

Note: kube-lego is only in maintenance mode as it been replaced with cert-manager.

### external-dns

As mentioned, we will be automatically pushing and syncing dns records for our new feature branches in Cloudflare. For this we are using external-dns.

[github.com/helm/charts/tree/master/stable/external-dns](https://github.com/helm/charts/tree/master/stable/external-dns)

Installed with (provide your Cloudflare api key and email)
```
helm upgrade --install cloudflare-dns --set rbac.create=true,sources={ingress},logLevel=debug,provider=cloudflare,cloudflare.apiKey=SECRET_KEY,cloudflare.email=user@example.com stable/external-dns
```

### nginx-ingress

Nginx-ingress will server as a ingress gateway. It will consume certificates stored in Kubernetes secrets by kube-lego.

[github.com/helm/charts/tree/master/stable/nginx-ingress](https://github.com/helm/charts/tree/master/stable/nginx-ingress)

Installed with:

```
helm upgrade --install nginx-ingress --set controller.publishService.enabled=true stable/nginx-ingress
```

## Gitlab

As this example is about Gitlab-ci, we will leverage some gitlab features.

### Gitlab secrets (Environments)

We will store our GCP credentials json in environment GCLOUD_SERVICE_KEY. This allows us to push our images to GCR and deploy our applications to Kubernetes cluster. Next we need to add our gitlab token into secret CI_JOB_TOKEN, so we can push our temporary images into gitlab container registry, see [docs.gitlab.com/ee/user/profile/personal_access_tokens.html](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)

### Gitlab runners

Our pipeline jobs will be executed on gitlab runners. As we will use default shared runners, we don't have to configure anything. If we would want to use our own runners, we would have to prepare instances, install gitlab-runner on them and authorize them to our gitlab pipeline.

### Gitlab repository

All our application code and pipeline configuration will be stored in gitlab repository.
