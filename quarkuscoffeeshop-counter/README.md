## quarkuscoffeeshop-counter tekton pipeline
![quarkuscoffeeshop-counter](../images/quarkuscoffeeshop-counter.png)

## Deploy pipelines using kustomize
> You may fork this repo and make edit to the `application-deployment/store/quarkuscoffeeshop-counter/transformer-patches.yaml` in for GitOps or argocd
---
**Create Projects and configure permissions**
```
oc new-project quarkuscoffeeshop-cicd
oc new-project quarkuscoffeeshop-demo
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-demo:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-demo -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-demo
```
**Run the Kustomize command to deploy pipelines** 
```
kustomize build quarkuscoffeeshop-counter | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkuscoffeeshop-counter  -n quarkuscoffeeshop-demo
```

## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f quarkuscoffeeshop-counter/pvc/pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-counter/pvc/maven-source-pvc.yml
```

**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-counter/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-counter/resources/git-pipeline-resource.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-counter/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-counter/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
```
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-demo:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-demo -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-demo

oc project quarkuscoffeeshop-demo
oc create -f application-deployment/store/quarkuscoffeeshop-counter/quarkuscoffeeshop-counter.yaml  -n quarkuscoffeeshop-demo
```

**Update Enviornment Variables in deployment**
```
oc edit deployment.apps/quarkuscoffeeshop-counter
```