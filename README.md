# Phase 1: Building Internal Developer Platform (IDP) with Red Hat Developer Hub

**Note Phase 1 is intended to show manuall setup of Keycloak by using `oc apply -f ...`commands**  
**Phase 2 uses ArgoCD to deploy the same resources**

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