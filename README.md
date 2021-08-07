# GitOps Mono Repo

The purpose of this repository is to demonstrate bootstrapping, configuring, and deploying applications to multiple OpenShift clusters, where each cluster may have multiple environments and have a different set of tools deployed as well.

The struture of this repository is heavily influenced by the [GitOps Standards](https://github.com/gnunn-gitops/standards) document authored by [Gerald Nunn](https://github.com/gnunn1).

The reason I have gone with a ["mono repo" approach](https://blog.argoproj.io/5-gitops-best-practices-d95cb0cbe9ff) since this is primarly for my own environments, demos, and experimentation.  If you prefer splitting GitOps git repositories up to more closely align with your organizational or team structure, you can still take some inspiration from this setup and simply consider each of the main top-level directories as being it's own repository (and perhaps even one repository per "app").

## Setup and Configuration

This repository is *not* meant to be reproduced in environments other than my own.  Since I'm using [Bitnami Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets), any component in this demo that depends on a secret (e.g. image pull secret or GitHub token) will onl work if the Sealed Secrets operator is seeded with my key (which is not in this repository).

The following documents the steps I take to bootstrap and configure one of my own clusters.

## Bootstrap

Starting from a fresh cluster, there are two main configurations that need to be in place for the rest of the process:
* OpenShift GitOps Operator - Provides cluster-level Argo CD instance.
* Sealed Secrets namespace and serets - Pre-seed this namespace with a secret that contains the expected encryption certificate.

Login to the new cluster as a `cluster-admin` user and run:

```
$ oc apply -k 00-bootstrap/00-seed
```

This will:
* Install the OpenShift GitOps operator, which will in turn deploy the cluster-level Argo CD instance.
* Create the `sealed-secrets` namespace.
* Seed the `sealed-secrets` namespace with the expected `Secret` (not in the repo - excluded in the main `.gitignore`).

In a future release of the OpenShift GitOps operator, the default Argo CD instance will have an edge-terminated TLS route.  For now, if you would like this functionality (instead of accepting self-signed certificate warning), run the following command once Argo CD has finished deploying in the `openshift-gitops` namespace.

```
$ oc patch argocd openshift-gitops \
    -n openshift-gitops \
    --type=merge \
    -p='{"spec":{"server":{"insecure":true,"route":{"enabled":true,"tls":{"insecureEdgeTerminationPolicy":"Redirect","termination":"edge"}}}}}'
```

To find the default admin password for Argo CD using the cli, run:

```
$ oc get secret openshift-gitops-cluster \
    -n openshift-gitops \
    -o jsonpath='{.data.admin\.password}' | base64 -d
```