# quarkuscoffeeshop Tekton pipelines Guide

### Requirements 
* [Postgres Operator](https://github.com/quarkuscoffeeshop/quarkuscoffeeshop-helm/wiki#install-postgres-operator)
* AMQ Streams

**Once Postgres Operator Database is installed run the following below**
```
$ ansible-galaxy collection install community.kubernetes
$ ansible-galaxy install tosin2013.quarkus_cafe_demo_role
$ export DOMAIN=ocp4.example.com
$ export OCP_TOKEN=123456789
$ export POSTGRES_PASSWORD=123456789
$ export STORE_ID=ATLANTA
$ cat >deploy-quarkus-cafe.yml<<YAML
- hosts: localhost
  become: yes
  vars:
    openshift_token: ${OCP_TOKEN}
    openshift_url: https://api.${DOMAIN}:6443
    insecure_skip_tls_verify: true
    default_owner: ${USER}
    default_group: ${USER}
    project_namespace: quarkuscoffeeshop-demo # Change to quarkuscoffeeshop-homeoffice for back office deployments 
    delete_deployment: false
    skip_amq_install: false
    skip_configure_postgres: false
    skip_mongodb_operator_install: true
    skip_quarkuscoffeeshop_helm_install: true
    domain: ${DOMAIN}
    postgres_password: "${POSTGRES_PASSWORD}"
    storeid: ${STORE_ID}
  roles:
    - tosin2013.quarkus_cafe_demo_role
YAML
$ ansible-playbook  deploy-quarkus-cafe.yml
```

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




