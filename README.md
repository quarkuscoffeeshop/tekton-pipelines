# quarkuscoffeeshop Tekton pipelines Guide

**Install OpenShift Pipelines 4.6**
```
cat <<EOF | oc -n openshift-operators create -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-pipelines-operator-rh
spec:
  channel: ocp-4.6
  installPlanApproval: Automatic
  name: openshift-pipelines-operator-rh
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

**Install OpenShift Pipelines 4.5**
```
cat <<EOF | oc -n openshift-operators create -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-pipelines-operator-rh
spec:
  channel: ocp-4.5
  installPlanApproval: Automatic
  name: openshift-pipelines-operator-rh
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

**Install tkn cli**  
`on linux`
```
# Get the tar.xz
curl -LO https://github.com/tektoncd/cli/releases/download/v0.13.1/tkn_0.13.1_Linux_x86_64.tar.gz
# Extract tkn to your PATH (e.g. /usr/local/bin)
sudo tar xvzf tkn_0.13.1_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
```

`on mac`
```
brew install tektoncd-cli
```

**Create Quarkus Coffee Shop project**
```
oc new-project quarkuscoffeeshop-cicd
```

**Set quay credentials**  
```
$ cat quay-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: quay-auth-secret
data:
  .dockerconfigjson: changeinfo==
type: kubernetes.io/dockerconfigjson
```

**Create secret**
```
oc create -f quay-secret.yml --namespace=quarkuscoffeeshop-cicd
```

## quarkuscoffeeshop-homeoffice-ui tekton pipeline
**configure pvc**
```
oc -n quarkuscoffeeshop-cicd create -f quarkuscoffeeshop-homeoffice-ui/pvc/pvc.yml
```

**configure Tasks**
```
oc -n quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-homeoffice-ui/tektontasks/s2i-nodejs-task.yaml
oc -n quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-homeoffice-ui/tektontasks/openshift-client-task.yaml
oc -n  quarkuscoffeeshop-cicd create -f ./quarkuscoffeeshop-homeoffice-ui/tektontasks/pushImageToQuay.yaml
```

**configure Resources**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-homeoffice-ui/resources/image-pipeline-resource.yaml.in
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-homeoffice-ui/resources/git-pipeline-resource.yaml
```

**Create Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-homeoffice-ui/pipeline/deploy-pipeline.yaml
```

**Create GitHub Trigger for Pipeline**
```
oc -n quarkuscoffeeshop-cicd create -f ./triggerbinding-configs/webhook-roles.yaml
oc -n quarkuscoffeeshop-cicd create -f ./triggerbinding-configs/github-triggerbinding.yaml
WEBHOOK_SECRET="$(openssl rand -base64 12)"
oc -n quarkuscoffeeshop-cicd create secret generic webhook-secret --from-literal=secret=${WEBHOOK_SECRET}
echo ${WEBHOOK_SECRET} > saved-secert.txt
sed -i "s/<git-triggerbinding>/github-triggerbinding/" ./quarkuscoffeeshop-homeoffice-ui/webhook.yaml
sed -i "/ref: github-triggerbinding/d" ./quarkuscoffeeshop-homeoffice-ui/webhook.yaml
sed -i "s/- name: pipeline-binding/- name: github-triggerbinding/" ./quarkuscoffeeshop-homeoffice-ui/webhook.yaml
oc -n quarkuscoffeeshop-cicd create -f  ./quarkuscoffeeshop-homeoffice-ui/webhook.yaml
```

**Create HomeOffice Webhook**
```
oc -n quarkuscoffeeshop-cicd create route edge homeoffice-ui-webhook --service=el-homeoffice-ui-webhook --port=8080 --insecure-policy=Redirect
```


oc -n quarkuscoffeeshop-cicd  get route homeoffice-ui-webhook -o jsonpath='https://{.spec.host}'

## To-Do
* add trigger for pipeline