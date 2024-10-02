# IDP Platform Components

GitOps manifest for an Internal Developer Platform built on OpenShift

## Bootstrapping OpenShift GitOps

1. Provision OpenShift GitOps operator:

   `oc apply -k argocd/bootstrap/operator`

2. Once the OpenShift GitOps operator is installed, confgure the platform ArgoCD instance:

   `oc apply -k argocd/bootstrap/instance`

3. When all pods in the `openshift-gitops` namespace are ready, use the URL resolved by the ArgoCD Route to open the user interface and verify that you can log in.

   Note: By default, only the OpenShift user called `admin` has full ArgoCD privileges. You can adjust the list of user that should have full ArgoCD privileges on the cluster by editing `argocd/bootstrap/instance/gitops-admins-group.yaml`.

4. Copy the file `argocd/platform-root.example.yaml` to `argocd/platform-root.yaml` and customize the configuration. (see [Configuration Guide](#configuration-guide)) 

   Note: Since you will potentially configure sensitive values in `argocd/platform-root.yaml` like a Git token, etc., it is included in `.gitignore` so sensitive values are not accidentally pushed to the remote.

5. Provision the root ArgoCD Application:

   `oc apply -f argocd/platform-root.yaml -n openshift-gitops`

6. In the ArgoCD UI, you will see a set of Applications begin to sync. Once all Applications are in a synced and healthy, the platform is installed.

## Configuration Guide

Below are the critical configuration parameters that require user-provided values in `argocd/platform-root.yaml`.


| Key                   | Description                                 | Default Value                           | Notes                                                    |
|-----------------------|---------------------------------------------|-----------------------------------------|----------------------------------------------------------|
| `clusterRouterDomain` | wildcard domain for the OpenShift Router    | `apps.example.cluster.com`              |                                                          |
| `gitHost`             | GitHub service provider host                | `https://github.com`                    |                                                          |
| `gitOrg`              | GitHub organization                         | `contract-first-idp`                    | e.g. your fork                                           |
| `gitRef`              | Git branch, tag, or commit                  | `main`                                  | e.g. your feature branch                                 |
| `gitToken`            | Backstage GitHub authentication token       | `ghp_REPLACEME`                         | Settings -> Developer Settings -> Personal Access Tokens |
| `eclipseClientId`     | Github registered DevSpaces AppID           | `devSpaceAppID`                         | Settings -> Developer Settings -> OAuth Apps             |
| `eclipseClientSecret` | Github registered DevSpaces AppSecret       | `devSpaceAppSecret`                     | Settings -> Developer Settings -> OAuth Apps             |   

## User Guide

### Developer Hub User Credentials

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

### Setting config values for DevSpaces 

We need to setup Eclipse-che for installation. Bases on the git provider repo used, we can get the current certificate:

    `openssl s_client -connect github.com:443`

So we can replace in _charts/dev-spaces/templates/self-signed-cert-cm.yaml_ under _custom-git-ca-certificates.pem_, the section between:

    -----BEGIN CERTIFICATE-----

    -----END CERTIFICATE-----

Go to `Settings -> Applications -> Authorized Oauth Apps` in Github User Profile side for creating OAuth DevSpace Authorization, generating ClientID, ClientSecret.

Let set the URL as:

      https://devspaces.apps.cluster-{your-domain}/

Let set the callback URL as:

      https://devspaces.apps.cluster-{your-domain}/api/oauth/callback

Set _eclipseClientId_, and _eclipseClientSecret_, in _platform-root.yaml_ properly, then re-apply again.
