# Phase 1: Building Internal Developer Platform (IDP) with Red Hat Developer Hub

**Note Phase 1 is intended to show manuall setup of Keycloak by using `oc apply -f ...`commands**  
**And then later uses ArgoCD to deploy the same resources. Manual set up is required the first time**  
**It is highly reccomended to use a secrets manager, such as Vault to store tls certs, passwords and URLs**

- Generate certs for phase-1-work keycloak setup

```bash
openssl req -x509 -newkey rsa:2048 -nodes -keyout tls.key -out tls.crt -days 365 -subj "/CN=www.example.com"
```

- Encode for `keycloak/3-secret.yaml`

```bash
base64 -w0 < tls.crt
```
Note Output

```bash
base64 -w0 < tls.key
```

Note Output.

Copy the output to `keycloak/3-secret.yaml`

```yaml
data:
  tls.crt: ...
  tls.key: ...
```

- Be sure to update `keycloak/5-keycloak-instance.yaml` with the correct domain for your cluster

```yaml
 hostname:
    adminUrl: 'https://keycloak-rhdh-operator.apps.cluster-<domain>.dynamic.redhatworkshops.io'
    hostname: keycloak-rhdh-operator.apps.cluster-<domain>.dynamic.redhatworkshops.io
```

- Once deployed, Keylcload admin password can be found in the `demo-keycloak-instance-initial-admin` secret in ns `demo-project`

Save that for later use

**Note: You need to follow the above steps manually and apply yaml files 1-5 before continuing. Once you deploy these resources, there are values you will need for the next set of steps**

- Github configuration for `keycloak/6-keycloak-realm.yaml`

1. Navigate to https://github.com/settings/apps
2. Select `OAuth Apps` Tab
3. Click `New OAuth App` Button
4. Fill out the iformation with the following details

| Field                     | Value                                                                                                                | 
|---                        |:---:                                                                                                                 |
| Application name          | `Keycloak_Auth`                                                                                                      | 
| Homepage URL              | `https://keycloak-rhdh-operator.apps.cluster-<DOMAIN>.dynamic.redhatworkshops.io`                                    | 
| Authorization callback URL| `https://keycloak-rhdh-operator.apps.cluster-<DOMAIN>.dynamic.redhatworkshops.io/realms/rhdh/broker/github/endpoint` | 

5. Click `Register application`
6. Make note of your `Client ID` for later use
7. Click `Generate a new client secret` and save it for later use
8. Click `Update application`

- Use data from previous step to populate CRD `keycloak/6-keycloak-realm.yaml`

```yaml
        config:
          clientId: "<CLIENT ID>"
          clientSecret: "<CLIENT SECRET>"
```

apply 6

- Delete files 1-6 and redeploy everything with argoCD (GitOps)
```bash
oc delete -f /keycloak/
```

- Redeploy using argoCD

```bash
cd rhdh-idp/argocd
```

```bash
oc apply -f keycloak.yaml  
```
- Manually deploy rhdh resources (if you want)

- Install RHDH with the argoCD app

Once up and running, switch to `phase-2-work` branch for next steps

- Merge `phase-1-work` with `phase-2-work` branch before continuing

```bash
git merge phase-1-work   
                                         
Auto-merging rhdh/rhdh/5-app-config-rhdh.yaml
Auto-merging rhdh/rhdh/6-backstage.yaml
Merge made by the 'ort' strategy.
 README.md                                  | 93 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 argocd/keycloak.yaml                       |  2 +-
 argocd/rhdh.yaml                           |  2 +-
 keycloak/keycloak/3-secret.yaml            |  4 ++--
 keycloak/keycloak/5-keycloak-instance.yaml |  4 ++--
 keycloak/keycloak/6-keycloak-realm.yaml    | 12 ++++++------
 rhdh/rhdh/5-app-config-rhdh.yaml           | 12 ++++++------
 rhdh/rhdh/6-backstage.yaml                 |  4 ++--
 8 files changed, 113 insertions(+), 20 deletions(-)
 ```

 - Manual setup for phase 2

1. Apply `rhdh/3-5-serviceAccount.yaml`
2. Extract the Kubernetes service account token for RHDH to use to access the cluster

```bash
oc -n demo-project get secret rhdh-k8s-sa-token -o jsonpath="{.data.token}" | base64 
```

3. Use the output to set `K8_SA_TOKEN` in `rhdh/4-Secret.yaml`
4. Generate and get personal access token (classic) from GitHub

  - https://github.com/settings/tokens
  - Personal access tokens (classic)
  - Generate New token
  - New personal access token (classic)
  - call it `RHDHTOKEN` (can be an name).
  - Select scopes (may vary based on requirements)
    - `repo`
    - `workflow`
    - `write:packages`
    - `admin:org`
    - `admin:repo_hook`
    - `admin:org_hook`
5. Copy generated token to `GITHUB_TOKEN` in `rhdh/4-Secret.yaml`
6. Apply `rhdh/4-Secret.yaml`
7. Modify GUID and apply `rhdh/5-app-config-rhdh.yaml`
8. Apply `rhdh/5-dynamic-plugins.yaml`
9. Apply `rhdh/6-backstage.yaml`

Once up and running, you now have the ability to add templates

Example:
https://github.com/RedHatQuickCourses/RHDH_Golden_Path/blob/module2/all-templates.yaml

- Once the template has been imported, get an API key from https://home.openweathermap.org/api_keys (for the app demo)
