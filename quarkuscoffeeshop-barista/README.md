## quarkuscoffeeshop-barista tekton pipeline

![quarkuscoffeeshop-barista](../images/quarkuscoffeeshop-barista.png)

**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f quarkuscoffeeshop-barista/pvc/pvc.yml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-barista/pvc/maven-source-pvc.yml
```


**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/git-clone.yaml
oc -n quarkuscoffeeshop-cicd create -f ./common-functions/tasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./common-functions/tasks/maven.yaml
```

**Configure push image to quay task**
```
oc -n  quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-barista/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-barista/resources/git-pipeline-resource.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-barista/resources/image-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-barista/pipeline/deploy-pipeline.yaml
```


### Integration testing instructions 
```
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-demo:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-demo -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-demo

oc project quarkuscoffeeshop-demo
oc create -f application-deployment/store/quarkuscoffeeshop-barista/quarkuscoffeeshop-barista.yaml -n quarkuscoffeeshop-demo
```