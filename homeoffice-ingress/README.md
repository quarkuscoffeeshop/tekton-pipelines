# homeoffice-ingress tekton pipeline


## Deploy pipelines using kustomize
---
> You may fork this repo and make edit to the `application-deployment/homeoffice/homeoffice-ingress/transformer-patches.yaml` in for GitOps or argocd
**Create Projects**
```
oc new-project quarkuscoffeeshop-cicd
oc new-project quarkuscoffeeshop-homeoffice
```

**Run the kustomize command to deploy pipelines** 
```
kustomize build homeoffice-ingress | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/homeoffice-ingress
```

## Configure webhooks
---
**See triggerbinding-configs before going to next step**  
* [triggerbinding-configs](../triggerbinding-configs)

> **NOTE**: Every Git server has its own properties, but basically you want to provide the ingress url for our webhook and when the Git server should send the hook. E.g: push events, PR events, etc.

1. Go to your application repository on GitHub, eg: https://github.com/quarkuscoffeeshop/homeoffice-ingress
2. Click on `Settings` -> `Webhooks`
3. Create the following `Hook`
   1. `Payload URL`: Output of command `oc -n quarkuscoffeeshop-cicd  get route homeoffice-ui-webhook -o jsonpath='https://{.spec.host}'`
   2. `Content type`: application/json
   2. `Secret`: v3r1s3cur3 `cat saved-secert.txt`
   3. `Events`: Check **Push Events**, leave others blank
   4. `Active`: Check it
   5. `SSL verification`: Check  **Disable**
   6. Click on `Add webhook`

Create Webhook for HomeOffice Ingress
```
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-ingress/webhook.yaml
```

Create HomeOffice Ingress Webhook
```
oc -n quarkuscoffeeshop-cicd create route edge homeoffice-ui-webhook --service=el-homeoffice-ui-webhook --port=8080 --insecure-policy=Redirect

oc -n quarkuscoffeeshop-cicd  get route homeoffice-ui-webhook -o jsonpath='https://{.spec.host}'
```

## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f homeoffice-ingress/pvc/pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-ingress/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkuscoffeeshop-cicd create -f ./homeoffice-ingress/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-ingress/resources/git-pipeline-resource.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-ingress/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-ingress/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
```
oc new-project quarkuscoffeeshop-homeoffice
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-homeoffice:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-homeoffice -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-homeoffice

oc project quarkuscoffeeshop-homeoffice
oc create -f  application-deployment/homeoffice/homeoffice-ingress/ -n quarkuscoffeeshop-homeoffice
oc expose service/homeoffice-ingress -n quarkuscoffeeshop-homeoffice
```


**Update Environment Variables in deployment**
```
oc edit deployment.apps/homeoffice-ingress
```