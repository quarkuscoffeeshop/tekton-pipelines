# quarkuscoffeeshop Tekton pipelines Guide

### Requirements 
* [Postgres Operator](https://github.com/quarkuscoffeeshop/quarkuscoffeeshop-helm/wiki#install-postgres-operator)
* AMQ Streams

**Once Postgres Operator Database is installed run the following below**
```
$ curl -OL https://raw.githubusercontent.com/quarkuscoffeeshop/quarkuscoffeeshop-ansible/master/files/deploy-quarkuscoffeeshop-ansible.sh
$ chmod +x deploy-quarkuscoffeeshop-ansible.sh
$ NAMESPACE=quarkuscoffeeshop-homeoffice
$  sed -i "s/quarkuscoffeeshop-demo/${NAMESPACE}/g" deploy-quarkuscoffeeshop-ansible.sh
$ ./deploy-quarkuscoffeeshop-ansible.sh 
 Options:
  -d      Add domain 
  -o      OpenShift Token
  -p      Postgres Password
  -s      Store ID
  -h      Display this help and exit
  -r      Destroy coffeeshop 
  To deploy qaurkuscoffeeshop-ansible playbooks
  ./deploy-quarkuscoffeeshop-ansible.sh  -d ocp4.example.com -o sha-123456789 -p 123456789 -s ATLANTA
$  ./deploy-quarkuscoffeeshop-ansible.sh  -d ocp4.example.com -o sha-123456789 -p 123456789 -s ATLANTA
```

**Install OpenShift Pipelines 4.6**
```
cat <<EOF | oc -n openshift-operators create -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-pipelines-operator-rh
spec:
  channel: stable
  installPlanApproval: Automatic
  name: openshift-pipelines-operator-rh
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

**Install tkn cli**  
`on linux AMD 64`
```
# Get the tar.xz
curl -LO https://github.com/tektoncd/cli/releases/download/v0.18.0/tkn_0.18.0_Linux_x86_64.tar.gz
# Extract tkn to your PATH (e.g. /usr/local/bin)
sudo tar xvzf tkn_0.18.0_Linux_x86_64.tar.gz -C /usr/local/bin/ tkn
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
* [tekton hub](https://hub-preview.tekton.dev/) 
* [Sending messages using Incoming Webhooks](https://api.slack.com/messaging/webhooks)
```
# oc create -f event-notification/send-to-webhook-slack.yaml -n quarkuscoffeeshop-cicd
# WEBHOOKURL=https://hooks.slack.com/services/xxxxx/Xxxxxx
# cat >webhook-secret.yaml<<YAML
kind: Secret
apiVersion: v1
metadata:
  name: webhook-secret
stringData:
  url: ${WEBHOOKURL}
YAML
# oc create -f webhook-secret.yaml -n quarkuscoffeeshop-cicd
```

## HOME Office (Backoffice)
**quarkuscoffeeshop-homeoffice-ui tekton pipline**  
[quarkuscoffeeshop-homeoffice-ui](quarkuscoffeeshop-homeoffice-ui/README.md)

**homeoffice-backend tekton pipline**  
[homeoffice-backend](homeoffice-backend/README.md)

## Store front microservices  

**quarkuscoffeeshop-barista tekton pipline**  
[quarkuscoffeeshop-barista](quarkuscoffeeshop-barista/README.md)

**quarkuscoffeeshop-counter tekton pipline**  
[quarkuscoffeeshop-counter](quarkuscoffeeshop-counter/README.md)

**quarkuscoffeeshop-kitchen tekton pipline**  
[quarkuscoffeeshop-kitchen](quarkuscoffeeshop-kitchen/README.md)

**quarkuscoffeeshop-homeoffice-ui tekton pipline**   
[quarkuscoffeeshop-web](quarkuscoffeeshop-web/README.md)


**quarkuscoffeeshop-homeoffice-ui tekton pipline**   
[quarkuscoffeeshop-web](quarkuscoffeeshop-web/README.md)


## RHEL Edge Pipelines
**quarkuscoffeeshop-majestic-monolith tekton pipline**   
[quarkuscoffeeshop-majestic-monolith](quarkuscoffeeshop-majestic-monolith/README.md)
