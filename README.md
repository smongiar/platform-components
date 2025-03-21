# Contract-First IDP Platform Components

GitOps manifest for an Internal Developer Platform built on OpenShift

## Preparing your own Fork

1. Choose an existing GitHub organization or create a new one for your own fork of Contract-First IDP

2. Fork the following repositories into your organization:
   - [`platform-components`](https://github.com/contract-first-idp/platform-components) - the platform k8s manifest
   - [`software-templates`](https://github.com/contract-first-idp/software-templates) - Backstage scaffolder templates
   - [`developer-charts`](https://github.com/contract-first-idp/developer-charts) - shared default charts for scaffolder templates
   - [`demo-domain`](https://github.com/contract-first-idp/demo-domain) - root GitOps entrypoint for platform tenants
   - [`spectral-rules`](https://github.com/contract-first-idp/spectral-rules) - a linter rule set configuration for API specs

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

| Key | Description | Notes |
|-----|-------------|-------|
| `clusterRouterDomain` | Wildcard domain for the OpenShift Router | Default value is `apps.example.cluster.com` |  |
| `gitHost` | GitHub service provider host | Default value is `https://github.com`  |
| `gitOrg` | GitHub organization | Default value is `contract-first-idp` e.g. your fork |
| `gitRef` | Git branch, tag, or commit | Default value is `main` e.g. your feature branch |
| `gitToken` | Backstage GitHub authentication token |  Generate a "classic" GitHub personal access token <br/>Settings -> Developer Settings -> Personal Access Tokens |
| `devSpacesClientId` `devSpacesClientSecret` | GitHub client ID and secret for Dev Spaces | See [Configuring a GitHub App for DevSpaces](https://docs.redhat.com/en/documentation/red_hat_openshift_dev_spaces/3.16/html/administration_guide/configuring-devspaces#configuring-oauth-2-for-github)) <br/>Reference callback URL: `https://devspaces.apps.example.cluster.com/api/oauth/callback` |


## Platform Demo Guide

1. Browse to your Developer Hub instance URL which is managed by the Route in the `developer-hub` namespace.

2. A Keycloak Realm including some predenifed users and groups is preconfigured as part of the platform for demonstration of OIDC integration with an enterprise identity provider. Chose a persona to log in to the developer portal. (See [Developer Hub User Credentials](#developer-hub-user-credentials))

3. Go to `Create...` on the side nagivation menu in the portal a view an inventory of templates. First select and execute the `System Template` which will create namespaces and a GitOps entrypoint for your new System.

4. Verify that a new GitHub repository was created representing the new System, and accept the pull request made to the `demo-domain` repository.

   Note: The ArgoCD UI will eventually display a new Application, OpenShift will have new namespaces configured. You can speed the process up by refreshing the `demo-domain` application in the ArgoCD UI.

5. Now browse again to `Create...` in the Developer Hub UI and this time select and execute `OpenAPI Specification` template. Make sure it is associated with the System you created previously.

6.  Verify that a new GitHub repository was created representing the new API and accept the pull request made to the System repostory created previously.

7. Once ArgoCD reconciles the new API, there will be a new API Specification pipeline in your System's build namepace in OpenShift. Iterate on the scaffoled OpenAPI specification, commit your changes, and run the API Specification pipeline to publish your specification version.

8. Now you can run any of the other component templates and select your existing API Entity in the scaffolder form. Various producer and consumer implementations are be bootstrapped by leveraging the API specification.

9. Optionally, iterate on the component implementation and commit your changes to its Git repository as needed. 

10. Run the component pipeline and test your component instance in its System's dev namepace.

### Developer Hub User Credentials

The is configured for RH SSO authentication. The following users and groups will be created and can be used to login. Keycloak user and group entity discovery is enabled and should sync the users and groups in backstage.

| User | Member of Group | Password |
|------|-----------------|----------|
| `backstage-admin` | `backstage-admins` | `letmein` |
| `user-ops-1` | `operators` | `letmein` |
| `user-ops-2` | `operators` | `letmein` |
| `user-dev-1` | `developers` | `letmein` |
| `user-dev-2` | `developers` | `letmein` |



