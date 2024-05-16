# IDP Platform Components

GitOps manifest for an Internal Developer Platform built on OpenShift

## Bootstrapping OpenShift GitOps

1. Provision OpenShift GitOps operator:

   `oc apply -k argocd/bootstrap/operator`

   Note: You may need to repeat the above command depending on namespace creation timing.

1. Once the OpenShift GitOps operator is installed, confgure the platform ArgoCD instance:

   `oc apply -k argocd/bootstrap/instance`

1. When all pods in the `openshift-gitops` namespace are ready, use the URL resolved by the ArgoCD Route to open the user interface and verify that you can log in.

   Note: By default, only the OpenShift user called `admin` has full ArgoCD privileges. You can adjust the list of user that should have full ArgoCD privileges on the cluster by editing `argocd/bootstrap/instance/gitops-admins-group.yaml`.

1. Copy the file `argocd/platform-root.example.yaml` to `argocd/platform-root.yaml` and customize the configuration. (see [Configuration Guide](#configuration-guide)) 

   Note: Since you will potentially configure sensitive values in `argocd/platform-root.yaml` like a Git token, etc., it is included in `.gitignore` so sensitive values are not accidentally pushed to the remote.

1. Provision the root ArgoCD Application:

   `oc apply -f argocd/platform-root.yaml`

1. In the ArgoCD UI, you will see a set of Applications begin to sync. Some of them might fail at first, potentially causing a cascade, so you may need to re-sync several times when intially provisioning the platform components.

## Configuration Guide

Below are the critical configuration parameters that require user-provided values.

### Cluster Router Domain

Several platform component charts need to know the base domain of the OpenShift Router. (This demo assumes wildcard DNS.)

Example:

```yaml
  source:
    ...
    helm:
      parameters:
        - name: clusterRouterDomain
          value: apps.cluster-xxxxx.dynamic.redhatworkshops.io
```


### GitOps Source

There are many ArgoCD Applications deployed in this demo which reference git repositories. You may be working in a fork of this repository, and perhaps in a feature branch. You can configure the location of your GitHub organization as well as your target git ref in one place.

Example:

```yaml
  source:
    ...
    helm:
      parameters:
        - name: gitHost
          value: https://github.com
        - name: gitOrg
          value: ghuser01
        - name: gitRef
          value: some-feature

```

### Backstage Git Token (GitHub Configuration)

In order to read GitHub URLs, Backstage needs an GitHub access token provided. An access token can be generated using the GitHub UI under `Settings -> Developer Settings -> Personal Access Tokens`.

Example:

```yaml
  source:
    ...
    helm:
      parameters:
        - name: "gitToken"
          value: ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

## Demo Guide

### User Authentication to the Developer Hub instance

This setup is configured for RH SSO authentication.  The following users and groups will be created and can be used to login.   Keycloak user and group entity discovery is enabled and should sync the users and groups in backstage.

Groups:

- `backstage-admins`
- `operators`
- `developers`

Users:

- `backstage-admin`
- `user-ops-1`
- `user-ops-2`
- `user-dev-1`
- `user-dev-2`

All users have an intial password set to: `letmein` 