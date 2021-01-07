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

**configure slack webhook**
```
oc create -f send-to-webhook-slack.yaml
oc create -f webhook-secret.yaml
```
## quarkuscoffeeshop-homeoffice-ui tekton pipline 
[quarkuscoffeeshop-homeoffice-ui](quarkuscoffeeshop-homeoffice-ui/README.md)

## homeoffice-backend tekton pipline 
[homeoffice-backend](homeoffice-backend/README.md)


