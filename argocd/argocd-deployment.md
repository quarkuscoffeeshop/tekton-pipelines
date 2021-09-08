# Quarkus Coffeshop and Argocd


1. Install OpenShifit GitOps Operator
2. Install OpenShift Pipelines

**Login to argocd**
```
argoRoute=$(oc get route quarkuscoffeeshop-server -n quarkuscoffeeshop-cicd -o jsonpath='{.spec.host}')
argoUser=admin
argoPass=$(oc get secret/quarkuscoffeeshop-cluster -n quarkuscoffeeshop-cicd -o jsonpath='{.data.admin\.password}' | base64 -d)
until [[ $(curl -ks -o /dev/null -w "%{http_code}"  https://${argoRoute}) -eq 200 ]]
do
    sleep 3
    echo -n '.'
done
argocd login --insecure --grpc-web --username ${argoUser} --password ${argoPass} ${argoRoute}
```

**This gives the serviceAccount for ArgoCD the ability to manage the cluster.**
```
oc adm policy add-cluster-role-to-user cluster-admin -z quarkuscoffeeshop-argocd-application-controller -n quarkuscoffeeshop-cicd 
oc adm policy add-cluster-role-to-user cluster-admin -z quarkuscoffeeshop-argocd-application-controller -n quarkuscoffeeshop-integration
```

```
argocd cluster add $(oc config current-context) --name=quarkuscoffeeshop-cicd --in-cluster --system-namespace=quarkuscoffeeshop-cicd --namespace=quarkuscoffeeshop-cicd
argocd cluster add $(oc config current-context) --name=quarkuscoffeeshop-cicd --in-cluster --system-namespace=quarkuscoffeeshop-integration --namespace=quarkuscoffeeshop-integration
```


**Export Enviornment variables**
```
export NAMESPACE=quarkuscoffeeshop-cicd
export GIT_URL=http://gogs-demo.apps.edgetesting.sandbox1007.opentlc.com
export GIT_USER=tosin
export ARGOCD_PROJECT_NAME=coffee-shop
export APP_NAME=quarkuscoffeeshop-majestic-monolith
```

**Add repo to  cluster**
```
argocd proj create ${ARGOCD_PROJECT_NAME} \
  --dest https://kubernetes.default.svc,${NAMESPACE} \
  --src ${GIT_URL}/${GITUSER}/tekton-pipelines.git \
  --description "ArgoCD project for ${APP_NAME} (EDGE)"
```

**Add repo**
```
argocd repo add ${GIT_URL}/${GIT_USER}/tekton-pipelines --username ${GIT_USER} --password COmp12#$  --name ${ARGOCD_PROJECT_NAME} 
```

**Add the project ${OCP_USER}-pipeline as a valid destination**
```
argocd proj add-destination ${APP_NAME} https://kubernetes.default.svc ${NAMESPACE}
```

**Create Deployment**
```
argocd app create ${ARGOCD_PROJECT_NAME} \
  --project ${ARGOCD_PROJECT_NAME} \
  --repo ${GIT_URL}/${GIT_USER}/tekton-pipelines.git \
  --path ${APP_NAME} \
  --revision HEAD \
  --dest-namespace ${NAMESPACE} \
  --dest-server https://kubernetes.default.svc
```

**Create Gogs project**
```
oc new-project gogs
oc new-app -f https://raw.githubusercontent.com/tosin2013/gogs-openshift-docker/master/openshift/gogs-persistent-template.yaml --param=HOSTNAME=gogs-demo.apps.edgetesting.sandbox1007.opentlc.com
```

oc extract secret/quarkuscoffeeshop-cluster -n quarkuscoffeeshop-cicd --to=-

```
argocd proj create quarkuscoffeeshop-majestic-monolith -d https://kubernetes.default.svc,quarkuscoffeeshop-cicd -s http://gogs-demo.apps.edgetesting.sandbox1007.opentlc.com/tosin/tekton-pipelines.git
```








```
oc adm policy add-cluster-role-to-user cluster-admin -z quarkuscoffeeshop-argocd-application-controller -n quarkuscoffeeshop-integration
```

```
 argocd cluster add $(oc config current-context) --name=quarkuscoffeeshop-cicd --in-cluster --system-namespace=quarkuscoffeeshop-cicd --namespace=quarkuscoffeeshop-cicd
```
 


 argocd cluster list
 source <(argocd completion bash)
