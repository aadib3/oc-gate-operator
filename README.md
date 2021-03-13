# Steps to deploy oc-gate operator on OCP cluster

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

## 4- Set the following variables with the appropriate image locations:
$ kuberbacproxyimage=pool6-infra1.practice.redhat.com:9446/kubebuilder/kube-rbac-proxy:v0.5.0

$ ocgateoperatorimage=quay.io/yaacov/oc-gate-operator@sha256:aa4b164d92372011e3c644651220f889671b8b4affc4b90a1c21eb4b10c84b60

## 5- Inject the image variables into oc-gate-operator.yaml file and create oc-gate-operator objects:
$ sed -i "s|kuberbacproxyimage|$kuberbacproxyimage|g;s|ocgateoperatorimage|$ocgateoperatorimage|g" oc-gate-operator.yaml

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

# Steps to authenticate access to a virtual machine noVNC console

## 1- Create a new secret oc-gate-jwt-secret in the oc-gate project:
$ oc create secret generic oc-gate-jwt-secret --from-file=certs/cert.pem --from-file=certs/key.pem -n oc-gate
``` bash
secret/oc-gate-jwt-secret created
```
## 2- Set the following variables required for creating the operator CRs:
``` bash
$ ocgateimage=quay.io/yaacov/oc-gate@sha256:ff929ae9ea5610e9fba6914485d7486e11f6d793685631e73541447d6c25f98c
$ ocgateroute="oc-gate.apps.ocp4.xxx.xxx"
$ vm=rhel6-150.ocp4.xxx.xxx 
$ ns=ocs-cnv
$ ocgatepath=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/$ns/virtualmachineinstances/$vm/vnc
$ posturl=https://$ocgateroute/login.html
$ postpath=/noVNC/vnc_lite.html?path=$ocgatepath
```

## 3- Inject the ocgateimage and ocgateroute variables into gateserver.yaml and create the GateServer custom resource:
$ sed -i "s|ocgateimage|$ocgateimage|g;s|ocgateroute|$ocgateroute|g" gateserver.yaml

$ oc create -f gateserver.yaml
``` bash
gateserver.ocgate.yaacov.com/oc-gate-server created
```

## 4- Inject the ocgateimage and ocgatepath into gatetoken.yaml and create the GateToken custom resource:
$ sed -i "s|ocgateimage|$ocgateimage|g;s|ocgatepath|$ocgatepath|g" gatetoken.yaml

$ oc create -f gatetoken.yaml
``` bash
gatetoken.ocgate.yaacov.com/oc-gate-token created
```

## 5- View custom resources created in oc-gate project:
$ oc get gateserver,gatetoken,po,svc,route -n oc-gate
``` bash
NAME                                          AGE
gateserver.ocgate.yaacov.com/oc-gate-server   3m2s

NAME                                        AGE
gatetoken.ocgate.yaacov.com/oc-gate-token   2m34s

NAME                                  READY   STATUS    RESTARTS   AGE
pod/oc-gate-server-5b957497d6-2cqsq   1/1     Running   0          3m1s

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/oc-gate-server   ClusterIP   172.30.204.235   <none>        8080/TCP   3m2s

NAME                                      HOST/PORT                       PATH   SERVICES         PORT   TERMINATION   WILDCARD
route.route.openshift.io/oc-gate-server   oc-gate.apps.ocp4.xxx.xxx          oc-gate-server   8080   reencrypt     None
```


## 6- Display content of the posturl variable, gatetoken object and postpath:
$ echo $posturl
``` bash
https://oc-gate.apps.ocp4.xxx.xxx/login.html
```
$ oc describe gatetoken oc-gate-token -n oc-gate | grep Token: | awk '{print $2}'
``` bash
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MTU1OTYzMDQsIm1hdGNoTWV0aG9kIjoiR0VULE9QVElPTlMiLCJtYXRjaFBhdGgiOiIvazhzL2FwaXMvc3VicmVzb3VyY2VzLmt1YmV2aXJ0LmlvL3YxYWxwaGEzL25hbWVzcGFjZXMvb2NzLWNudi92aXJ0dWFsbWFjaGluZWluc3RhbmNlcy9yaGVsNi0xNTAub2NwNC5nb2xkbWFuLmxhYi92bmMiLCJuYmYiOjE2MTU1OTI3MDR9.TyfeVzu9RvY3iBs5VsDcP2q7Xhs8RZ1OHt1I5lBkA_p6Ul55ccdM0OPNsLdWDrEVMBv2EUnASwS4CnJnQkMDOhPhSVW1h-0Yh4nafhDq_pmIGxoz-t_7y98ou31gwjwI-5dRMX02t4h7NythpiYPvSnIqihB31J30dvgkeDohburW6xvucAKx2p9OXEE2sgOVrpkcTsmCFbMqCYYh6me8ay_FHH0Ncvpa2OYqi_i337QKn2-bQMRvpXzM6r0v4eWmveNB68sAnt_PyJO4NuJ3pvlYlmMeFYCqB7_2J0QyCe1C9GCj34Xsf6nLEwUAK6usCxLJjoH7XwXFPBWoHm9fA
```
$ echo $postpath
``` bash
/noVNC/vnc_lite.html?path=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/ocs-cnv/virtualmachineinstances/rhel6-150.ocp4.xxx.xxx/vnc
```

## 7- Goto to the posturl enter token value in token field and the postpath in the Then field, then click Submit:
![Screenshot from 2021-03-08 13-29-39](https://user-images.githubusercontent.com/77073889/110364968-6eb42b00-8012-11eb-92f0-cabe751ec733.png)
![Screenshot from 2021-03-08 13-32-08](https://user-images.githubusercontent.com/77073889/110365266-d4a0b280-8012-11eb-8a89-26bd1d58be21.png)
