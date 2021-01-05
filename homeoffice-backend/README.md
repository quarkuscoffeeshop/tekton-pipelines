
## homeoffice-backend tekton pipeline

**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f homeoffice-backend/pvc/pvc.yml
```


**configure Tasks**
```
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
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-homeoffice-ui/pipeline/deploy-pipeline.yaml
```

#oc -n quarkuscoffeeshop-cicd create -f ./homeoffice-backend/tektontasks/s2i-quarkus-task.yaml


oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-backend/pvc/maven-source-pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./homeoffice-backend/pipelines/deploy-pipeline.yaml