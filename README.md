# Contract-First IDP Platform Components

GitOps manifest for an Internal Developer Platform built on OpenShift

## Preparing your own Fork

1. Choose an existing GitHub organization or create a new one for your own fork of Contract-First IDP

2. Fork the following repositories into your organization:
   - [`platform-components`](https://github.com/contract-first-idp/platform-components) - the platform k8s manifest
   - [`software-templates`](https://github.com/contract-first-idp/software-templates) - Backstage scaffolder templates
   - [`developer-charts`](https://github.com/contract-first-idp/developer-charts) - shared default charts for scaffolder templates
   - [`demo-domain`](https://github.com/contract-first-idp/demo-domain) - root GitOps entrypoint for platform tenants

3. Replace every instance of the string `contract-first-idp` in your `software-templates` fork with your GitHub organization name in your fork and commit that change to the main branch

4. Replace every instance of the string `apps.cluster-example.com` in your `software-templates` fork with your cluster's application router wildcard domain in your fork and commit that change to the main branch

## Bootstrapping OpenShift GitOps

1. Provision OpenShift GitOps operator:

   `oc apply -k argocd/bootstrap/operator`

2. Once the OpenShift GitOps operator is installed, confgure the platform ArgoCD instance:

   `oc apply -k argocd/bootstrap/instance`

3. When all pods in the `openshift-gitops` namespace are ready, use the URL resolved by the ArgoCD Route to open the user interface and verify that you can log in.

   Note: By default, only the OpenShift user called `admin` has full ArgoCD privileges. You can adjust the list of user that should have full ArgoCD privileges on the cluster by editing `argocd/bootstrap/instance/gitops-admins-group.yaml`.

## Provisioning the Developer Platform

1. Copy the file `argocd/platform-root.example.yaml` to `argocd/platform-root.yaml`.

2. Adjust the value of `source.repoUrl` in `argocd/platform-root.example.yaml` to point to your fork of the `platform-components` repository.

3. Customize the override values in `argocd/platform-root.yaml`. (see [Platform Configuration Guide](#platform-configuration-guide)) 

   Note: Since you will potentially configure sensitive values in `argocd/platform-root.yaml` like a Git token, etc., it is included in `.gitignore` so sensitive values are not accidentally pushed to the remote.

4. Provision the root ArgoCD Application:

   `oc apply -f argocd/platform-root.yaml -n openshift-gitops`

5. In the ArgoCD UI, you will see a set of Applications begin to sync. Once all Applications are in a synced and healthy, the platform is installed.

### Platform Configuration Guide

| Key | Description | Default Value | Notes |
|-----|-------------|---------------|-------|
| `clusterRouterDomain` | wildcard domain for the OpenShift Router | `apps.example.cluster.com` |  |
| `gitHost` | GitHub service provider host | `https://github.com` |  |
| `gitOrg` | GitHub organization | `contract-first-idp` | e.g. your fork |
| `gitRef` | Git branch, tag, or commit | `main` | e.g. your feature branch |
| `gitToken` | Backstage GitHub authentication token | `ghp_REPLACEME` | Settings -> Developer Settings -> Personal Access Tokens |
| `serviceAccountToken` | Backstage k8s authentication token | `eyREPLACEME` | [See Backstage K8s Integration Docs](https://backstage.io/docs/features/kubernetes/configuration#clustersserviceaccounttoken-optional)
| `eclipseClientId` | Github registered DevSpaces AppID | `devSpaceAppID` | Settings -> Developer Settings -> OAuth Apps |
| `eclipseClientSecret` | Github registered DevSpaces AppSecret | `devSpaceAppSecret` | Settings -> Developer Settings -> OAuth Apps |

### Configuring a GitHub App for DevSpaces 

Go to `Settings -> Applications -> Authorized Oauth Apps` in Github User Profile side for creating OAuth DevSpace Authorization, generating ClientID, ClientSecret.

Set the URL as:

      https://devspaces.apps.cluster-{your-domain}/

Set the callback URL as:

      https://devspaces.apps.cluster-{your-domain}/api/oauth/callback

## Platform User Guide

...

### Developer Hub User Credentials

The is configured for RH SSO authentication. The following users and groups will be created and can be used to login. Keycloak user and group entity discovery is enabled and should sync the users and groups in backstage.

| User | Member of Group | Password |
|------|-----------------|----------|
| `backstage-admin` | `backstage-admins` | `letmein` |
| `user-ops-1` | `operators` | `letmein` |
| `user-ops-2` | `operators` | `letmein` |
| `user-dev-1` | `developers` | `letmein` |
| `user-dev-2` | `developers` | `letmein` |



