# Quarkus Coffeshop and Argocd

## Install the ACM_WORKLOADS option from the deploy-quarkuscoffeeshop-ansible.sh

```
$ curl -OL https://raw.githubusercontent.com/quarkuscoffeeshop/quarkuscoffeeshop-ansible/dev/files/deploy-quarkuscoffeeshop-ansible.sh
$ chmod +x deploy-quarkuscoffeeshop-ansible.sh
```
## The script will provide the following
* Gogs server
* OpenShift Pipelines
* OpenShift GitOps
* Quay.io
* AMQ Streams
* Postgres Template deployment
* homeoffice Tekton pipelines
* quarkus-coffeeshop Tekton pipelines
```
$ cat >env.variables<<EOF
ACM_WORKLOADS=y
AMQ_STREAMS=y
CONFIGURE_POSTGRES=y
MONGODB_OPERATOR=n
MONGODB=n
HELM_DEPLOYMENT=n
EOF
$ ./deploy-quarkuscoffeeshop-ansible.sh -d ocp4.example.com -t sha-123456789 -p 123456789 -s ATLANTA
```

**Install argocd cli**
```
curl -OL https://github.com/argoproj/argo-cd/releases/download/v2.1.2/argocd-linux-amd64 
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
sudo chmod +x /usr/local/bin/argocd
```

**Login to argocd**
```
nameSpace=openshift-gitops
argoRoute=$(oc get route openshift-gitops-server -n ${nameSpace} -o jsonpath='{.spec.host}')
argoUser=admin
argoPass=$(oc get secret/openshift-gitops-cluster -n ${nameSpace} -o jsonpath='{.data.admin\.password}' | base64 -d)
until [[ $(curl -ks -o /dev/null -w "%{http_code}"  https://${argoRoute}) -eq 200 ]]
do
    sleep 3
    echo -n '.'
done
argocd login --insecure --grpc-web --username ${argoUser} --password ${argoPass} ${argoRoute}

echo "$argoRoute"
echo "$argoPass"
```



## Configure REPO URL
Fork [tekton-pipelines](https://github.com/quarkuscoffeeshop/tekton-pipelines.git) and update REPO_URL.
```
$ export REPO_URL='https://github.com/quarkuscoffeeshop/tekton-pipelines.git'
# Example Alternative Repo
REPO_URL='http://gogs-quarkuscoffeeshop-cicd.apps.cluster-e6dd.e6dd.sandbox568.opentlc.com/user1/tekton-pipelines.git'
```

## HOME Office (Backoffice)
**quarkuscoffeeshop-homeoffice-ui argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkuscoffeeshop-homeoffice-ui/quarkuscoffeeshop-homeoffice-ui-template.yaml  > argocd/quarkuscoffeeshop-homeoffice-ui/quarkuscoffeeshop-homeoffice-ui.yaml
oc create -f argocd/quarkuscoffeeshop-homeoffice-ui/quarkuscoffeeshop-homeoffice-ui.yaml  -n openshift-gitops
```

**homeoffice-backend argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/homeoffice-backend/homeoffice-backend-template.yaml  > argocd/homeoffice-backend/homeoffice-backend.yaml
oc create -f argocd/homeoffice-backend/homeoffice-backend.yaml  -n openshift-gitops
```

**homeoffice-ingress argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/homeoffice-ingress/homeoffice-ingress-template.yaml  > argocd/homeoffice-ingress/homeoffice-ingress.yaml
oc create -f argocd/homeoffice-ingress/homeoffice-ingress.yaml  -n openshift-gitops
```

## Store front microservices  

**quarkuscoffeeshop-barista argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkuscoffeeshop-barista/quarkuscoffeeshop-barista-template.yaml  > argocd/quarkuscoffeeshop-barista/quarkuscoffeeshop-barista.yaml
oc create -f argocd/quarkuscoffeeshop-barista/quarkuscoffeeshop-barista.yaml  -n openshift-gitops
```

**quarkuscoffeeshop-counter argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkuscoffeeshop-counter/quarkuscoffeeshop-counter-template.yaml  > argocd/quarkuscoffeeshop-counter/quarkuscoffeeshop-counter.yaml
oc create -f argocd/quarkuscoffeeshop-counter/quarkuscoffeeshop-counter.yaml  -n openshift-gitops
```

**quarkuscoffeeshop-kitchen argo application**  
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkuscoffeeshop-kitchen/quarkuscoffeeshop-kitchen-template.yaml  > argocd/quarkuscoffeeshop-kitchen/quarkuscoffeeshop-kitchen.yaml
oc create -f argocd/quarkuscoffeeshop-kitchen/quarkuscoffeeshop-kitchen.yaml  -n openshift-gitops
```

**quarkuscoffeeshop-web argo application**   
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkuscoffeeshop-web/quarkuscoffeeshop-web-template.yaml  > argocd/quarkuscoffeeshop-web/quarkuscoffeeshop-web.yaml
oc create -f argocd/quarkuscoffeeshop-web/quarkuscoffeeshop-web.yaml  -n openshift-gitops
```

## RHEL Edge Pipelines
**quarkuscoffeeshop-majestic-monolith argo application**   
```
sed "s|%REPO_NAME%|'${REPO_URL}'|g" argocd/quarkuscoffeeshop-monolith/quarkuscoffeeshop-monolith-template.yaml  > argocd/quarkuscoffeeshop-monolith/quarkuscoffeeshop-monolith.yaml
oc create -f argocd/quarkuscoffeeshop-monolith/quarkuscoffeeshop-monolith.yaml  -n openshift-gitops
```


