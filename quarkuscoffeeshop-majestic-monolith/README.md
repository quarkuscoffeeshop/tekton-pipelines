# quarkuscoffeeshop-majestic-monolith tekton pipeline

## Deploy pipelines using kustomize
> You may fork this repo and make edit to the `application-deployment/store/quarkuscoffeeshop-majestic-monolith/transformer-patches.yaml` in for GitOps or argocd
---
**Create Projects and configure permissions**
```
oc new-project quarkuscoffeeshop-cicd
oc new-project quarkuscoffeeshop-integration
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-integration:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-integration -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-integration
```
**Run the kustomize command to deploy pipelines** 
```
kustomize build quarkuscoffeeshop-majestic-monolith | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkuscoffeeshop-majestic-monolith  -n quarkuscoffeeshop-integration
```

## Configure webhooks
---
**See triggerbinding-configs before going to next step**  
* [triggerbinding-configs](../triggerbinding-configs)

Create Webhook for quarkuscoffeeshop-majestic-monolith
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-majestic-monolith/webhook.yaml
```

Create quarkuscoffeeshop-majestic-monolith Webhook
```
oc -n quarkuscoffeeshop-cicd create route edge monolith-webhook --service=el-quarkuscoffeeshop-majestic-monolith-webhook --port=8080 --insecure-policy=Redirect
```

> **NOTE**: Every Git server has its own properties, but basically you want to provide the ingress url for our webhook and when the Git server should send the hook. E.g: push events, PR events, etc.

1. Go to your application repository on GitHub, eg: https://github.com/jeremyrdavis/quarkuscoffeeshop-majestic-monolith
2. Click on `Settings` -> `Webhooks`
3. Create the following `Hook`
   1. `Payload URL`: Output of command `oc -n quarkuscoffeeshop-cicd  get route monolith-webhook -o jsonpath='https://{.spec.host}'`
   2. `Content type`: application/json
   2. `Secret`: v3r1s3cur3 `cat saved-secert.txt`
   3. `Events`: Check **Push Events**, leave others blank
   4. `Active`: Check it
   5. `SSL verification`: Check  **Disable**
   6. Click on `Add webhook`


## Deploy pipelines Manually 
---

**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f quarkuscoffeeshop-majestic-monolith/pvc/pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-majestic-monolith/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-majestic-monolith/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-majestic-monolith/resources/git-pipeline-resource.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-majestic-monolith/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-majestic-monolith/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
1. Configure permissions for project
```
oc new-project quarkuscoffeeshop-integration
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-integration:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-integration -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-integration
```
2. Run build pipeline 
3. Run the below commands 

```
oc project quarkuscoffeeshop-integration
oc create -f application-deployment/store/quarkuscoffeeshop-majestic-monolith  -n quarkuscoffeeshop-integration
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkuscoffeeshop-majestic-monolith
```