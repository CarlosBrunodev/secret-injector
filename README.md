# IBM secret vault - injector

[Link Documentacao ](https://ibm.docs.delinea.com/online-help/products/isvp-devops-vault/current/integrations/kubernetes/kubernetes-webhook.md)


![arquitetura](./img/Screenshot%202023-10-26%20at%2015.35.48.png)

Kubernetes Mutating Webhook
The Kubernetes Mutating Webhook has two parts, the Injector and the Syncer.

The Injector uses a YAML definition that maps secrets in a DSV tenant to variables in the Kubernetes secrets area. It runs when the cluster starts, sets these variables, and populates them with the secrets data from DSV.

Then the Syncer runs as a cron task, generally every minute, that updates the Kubernetes environment with updates that happen in DSV.

## Install 

### Manually Creating (Prior Method Before Automation)

JSON Credentials for Helm Install

The configuration requires a JSON formatted list of Client Credential and Tenant mappings.

The name of the credential (such as app1 or default) is used for matching the annontated credential to the right credentials file to use to connect to the connect tenant.

You can place your temporary config in .cache/credentials.json as this is ignored by git, so that you can run the helm install command manually if you aren't doing local development.


``````
$ vim .cache/credentials.json
``````

````json
{
  "app1": {
    "credentials": {
      "clientId": "",
      "clientSecret": "xxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxxxx-xxxxx"
    },
    "tenant": "mytenant"
  },
  "default": {
    "credentials": {
      "clientId": "",
      "clientSecret": "xxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxxxxx-xxxxx"
    },
    "tenant": "mytenant"
  }
}
````

### Helm Install

Installation of charts into the a cluster requires Helm.

There are two separate charts for the dsv-injector and the dsv-syncer.

The dsv-injector chart imports credentials.json from the filesystem and stores it in a Kubernetes Secret.
The dsv-syncer chart refers to that Secret instead of creating its own.

````
NAMESPACE='testing'
CREDENTIALS_JSON_FILE='.cache/credentials.json'
IMAGE_REPOSITORY='docker.io/delineaxpm/dsv-k8s'

helm install
     --namespace $NAMESPACE
     --create-namespace \
     --set-file credentialsJson=${CREDENTIALS_JSON_FILE} \
      --set image.repository=${IMAGE_REPOSITORY} \
     dsv-injector ./charts/dsv-injector

helm install
     --namespace $NAMESPACE
     --create-namespace \
     --set-file credentialsJson=${CREDENTIALS_JSON_FILE} \
      --set image.repository=${IMAGE_REPOSITORY} \
     dsv-syncer ./charts/dsv-syncer
````

### Update Manifests

This would be referenced by a Kubernetes secret with annotations like:

`````yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: user-domain-pass
  annotations:
    dsv.delinea.com/credentials: app1
    dsv.delinea.com/set-secret: 'tests:dsv-k8s'
`````

### Service web 

````
    $ k port-forward svc/dsv-injector 8543:8543

    $ curl -v http://localhost:8543/
````