## quarkuscoffeeshop-customermocker tekton pipeline


## Deploy pipelines using kustomize
> You may fork this repo and make edit to the `application-deployment/store/quarkuscoffeeshop-customermocker/transformer-patches.yaml` in for GitOps or argocd
---
**Set target project**
```
export TARGET_PROJECT=quarkuscoffeeshop-demo
or 
export TARGET_PROJECT=quarkuscoffeeshop-integration
```

**Create Projects and configure permissions**
```
oc new-project quarkuscoffeeshop-cicd
oc new-project ${TARGET_PROJECT}
oc adm policy add-role-to-user admin system:serviceaccount:${TARGET_PROJECT}:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:${TARGET_PROJECT} -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n ${TARGET_PROJECT}
```

**Update the kustomization.yaml with TARAGE_PROJECT name**
```
 sed -i "s/^namespace:.*/namespace: ${TARGET_PROJECT}/" application-deployment/store/quarkuscoffeeshop-customermocker/kustomization.yaml
```

**Run the kustomize command to deploy pipelines** 
```
kustomize build quarkuscoffeeshop-customermocker | oc create -f - 
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkuscoffeeshop-customermocker  -n ${TARGET_PROJECT}
```


## Deploy pipelines Manually 
---
**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f quarkuscoffeeshop-customermocker/pvc/pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-customermocker/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-customermocker/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-customermocker/resources/git-pipeline-resource.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-customermocker/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-customermocker/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 

**Set target project**
```
export TARGET_PROJECT=quarkuscoffeeshop-demo
or 
export TARGET_PROJECT=quarkuscoffeeshop-integration
```

**Create Projects and configure permissions**
```
oc new-project quarkuscoffeeshop-cicd
oc new-project ${TARGET_PROJECT}
oc adm policy add-role-to-user admin system:serviceaccount:${TARGET_PROJECT}:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:${TARGET_PROJECT} -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n ${TARGET_PROJECT}
```

**Deploy application**
```
oc project quarkuscoffeeshop-demo
oc create -f application-deployment/store/quarkuscoffeeshop-customermocker/ -n quarkuscoffeeshop-demo
oc expose service/quarkuscoffeeshop-customermocker -n quarkuscoffeeshop-demo
```