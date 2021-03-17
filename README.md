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

## 2 - Create certs dirs in cloned dir and populate certs with SSL certs:
$ cd oc-gate-operator

$ mkdir certs

$ openssl genrsa -out certs/key.pem
``` bash
Generating RSA private key, 2048 bit long modulus (2 primes)
..............+++++
..............................+++++
e is 65537 (0x010001)
$
```

$ openssl req -new -x509 -sha256 -key certs/key.pem -out certs/cert.pem -days 3650
``` bash
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [US]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:
$
```

$ ls certs
``` bash
cert.pem  key.pem
$
```

## 3- Set the following variables with the appropriate image locations:
$ kuberbacproxyimage=pool6-infra1.practice.redhat.com:9446/kubebuilder/kube-rbac-proxy:v0.5.0

$ ocgateoperatorimage=pool6-infra1.practice.redhat.com:9446/yaacov/oc-gate-operator

$ ocgateimage=pool6-infra1.practice.redhat.com:9446/yaacov/oc-gate

$ ocgatewebimage=pool6-infra1.practice.redhat.com:9446/yaacov/oc-gate-web-app-novnc

$ ocgateroute=oc-gate.apps.ocp4.xxx.xxx


## 4- Login into OCP cluster to deploy oc-gate-operator:
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

## 5- Inject the image variables into oc-gate-operator.yaml file and create oc-gate-operator objects:
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

## 6- View resources created in oc-gate-operator project:
$ oc get all -n oc-gate-operator
``` bash
NAME                                                      READY   STATUS    RESTARTS   AGE
pod/oc-gate-operator-controller-manager-566d6c44d-pmw7j   2/2     Running   1          10m

NAME                                                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/oc-gate-operator-controller-manager-metrics-service   ClusterIP   172.30.85.114   <none>        8443/TCP   10m

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oc-gate-operator-controller-manager   1/1     1            1           10m

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/oc-gate-operator-controller-manager-566d6c44d   1         1         1       10m
```

## 7- Inject the ocgateimage and ocgateroute variables into gateserver.yaml and create the GateServer custom resource:

$ sed -i "s|OCGATEIMAGE|$ocgateimage|g;s|OCGATEROUTE|$ocgateroute|g;s|OCGATEWEBIMAGE|$ocgatewebimage|g" gateserver.yaml

$ oc create -f gateserver.yaml
``` bash
gateserver.ocgate.yaacov.com/oc-gate-server created
```
