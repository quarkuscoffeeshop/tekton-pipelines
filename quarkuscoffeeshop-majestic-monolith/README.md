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