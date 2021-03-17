# Steps to deploy oc-gate operator on OCP cluster (One Time Setup)

## 1- Clone oc-gate-operator git repository:
``` bash
$ git clone https://github.com/aadib3/oc-gate-operator.git
Cloning into 'oc-gate-operator'...
remote: Enumerating objects: 29, done.
remote: Counting objects: 100% (29/29), done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 29 (delta 8), reused 29 (delta 8), pack-reused 0
Receiving objects: 100% (29/29), 11.93 KiB | 5.96 MiB/s, done.
Resolving deltas: 100% (8/8), done.
```


## 2- Set the following variables with the appropriate image locations:
$ kuberbacproxyimage=pool6-infra1.practice.redhat.com:9446/kubebuilder/kube-rbac-proxy:v0.5.0

$ ocgateoperatorimage=pool6-infra1.practice.redhat.com:9446/yaacov/oc-gate-operator

$ ocgateimage=pool6-infra1.practice.redhat.com:9446/yaacov/oc-gate

$ ocgatewebimage=pool6-infra1.practice.redhat.com:9446/yaacov/oc-gate-web-app-novnc

$ ocgateroute=oc-gate.apps.ocp4.xxx.xxx


## 3- Login into OCP cluster to deploy oc-gate-operator:
$ oc login https://api.ocp4.xxx.xxx:6443
``` bash
Authentication required for https://api.ocp4.xxx.xxx:6443 (openshift)
Username: xxxx
Password: 
Login successful.

You have access to xx projects, the list has been suppressed. You can list all projects with ' projects'

Using project "default".
$
```


## 4- Inject the image variables into oc-gate-operator.yaml file and create oc-gate-operator objects:
$ sed -i "s|KUBERBCPROXYIMAGE|$kuberbacproxyimage|g;s|OCGATEOPERATORIMAGE|$ocgateoperatorimage|g" oc-gate-operator.yaml

$ oc create -f oc-gate-operator.yaml
``` bash
namespace/oc-gate-operator created
namespace/oc-gate created
customresourcedefinition.apiextensions.k8s.io/gateservers.ocgate.yaacov.com created
customresourcedefinition.apiextensions.k8s.io/gatetokens.ocgate.yaacov.com created
role.rbac.authorization.k8s.io/oc-gate-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/oc-gate-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/oc-gate-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/oc-gate-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/oc-gate-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/oc-gate-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/oc-gate-operator-proxy-rolebinding created
configmap/oc-gate-operator-manager-config created
service/oc-gate-operator-controller-manager-metrics-service created
deployment.apps/oc-gate-operator-controller-manager created
```


## 5- Inject the ocgateimage and ocgateroute variables into gateserver.yaml and create the GateServer custom resource:

$ sed -i "s|OCGATEIMAGE|$ocgateimage|g;s|OCGATEROUTE|$ocgateroute|g;s|OCGATEWEBIMAGE|$ocgatewebimage|g" gateserver.yaml

$ oc create -f gateserver.yaml
``` bash
gateserver.ocgate.yaacov.com/oc-gate-server created
```
