## quarkuscoffeeshop-majestic-monolith tekton pipeline


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
oc new-project quarkuscoffeeshop-rhel-edge
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-rhel-edge:pipeline -n quarkuscoffeeshop-cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:quarkuscoffeeshop-rhel-edge -n quarkuscoffeeshop-cicd
oc adm policy add-role-to-user admin system:serviceaccount:quarkuscoffeeshop-cicd:pipeline -n quarkuscoffeeshop-rhel-edge
```
2. Run build pipeline 
3. Run the below commands 

```
oc project quarkuscoffeeshop-rhel-edge
oc create -f application-deployment/store/quarkuscoffeeshop-majestic-monolith/quarkuscoffeeshop-majestic-monolith.yaml -n quarkuscoffeeshop-rhel-edge
oc expose svc/quarkuscoffeeshop-majestic-monolith    
```

**Update Environment Variables in deployment**
```
oc edit deployment.apps/quarkuscoffeeshop-majestic-monolith
```