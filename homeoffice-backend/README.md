
## homeoffice-backend tekton pipeline

**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f homeoffice-backend/pvc/pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-backend/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./homeoffice-backend/tektontasks/pushImageToQuay.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./homeoffice-backend/tektontasks/maven.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-backend/resources/git-pipeline-resource.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-backend/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-backend/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
```
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-homeoffice:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-homeoffice -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-homeoffice

oc project quarkuscoffeeshop-homeoffice
oc new-app quarkuscoffeeshop-cicd/homeoffice-backend:latest -n quarkuscoffeeshop-homeoffice
oc expose service/homeoffice-backend -n quarkuscoffeeshop-homeoffice
```