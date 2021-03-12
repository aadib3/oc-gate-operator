# Steps to deploy oc-gate operator on OCP cluster

## 1 - Create oc-gate-operator/certs dirs and populate certs with SSL certs:
$ mkdir -p oc-gate-operator/certs

$ cd oc-gate-operator

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

## 2- Login into OCP cluster to deploy oc-gate app:
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

## 3- Create oc-gate-operator project:
$ oc new-project oc-gate-operator
``` bash
Now using project "oc-gate-operator" on server "https://api.xxx.xxx.lab:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname
$
```

## 4- Create a new secret oc-gate-jwt-secret in the oc-gate project:
$ oc create secret generic oc-gate-jwt-secret --from-file=certs/cert.pem
``` bash
secret/oc-gate-jwt-secret created
```

## 5- Create oc-gate template using included oc-gate-template.yaml:
$ oc create -f oc-gate-template.yaml
``` bash
template.template.openshift.io/oc-gate created
```

## 6- Create oc-gate OCP objects using the oc-gate template:
$ oc process -p ROUTE_URL=oc-gate.apps.ocp4.xxx.xxx oc-gate | oc create -f -
``` bash
route.route.openshift.io/oc-gate created
serviceaccount/oc-gate created
clusterrolebinding.authorization.openshift.io/oc-gate-cluster-reader created
service/oc-gate created
replicationcontroller/oc-gate created
```

## 7- Verify OCP objects created and running:
$ oc get all
``` bash
NAME                READY   STATUS    RESTARTS   AGE
pod/oc-gate-rx4p4   1/1     Running   0          2m24s

NAME                            DESIRED   CURRENT   READY   AGE
replicationcontroller/oc-gate   1         1         1       2m25s

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/oc-gate   ClusterIP   172.30.140.240   <none>        8080/TCP   2m25s

NAME                               HOST/PORT                       PATH   SERVICES   PORT   TERMINATION   WILDCARD
route.route.openshift.io/oc-gate   oc-gate.apps.ocp4.xxx.xxx          oc-gate    8080   reencrypt     None
```

# Steps to install golang and jwt
## 1- As root yum install golang:

\# yum install golang -y


## 2- Go get jwt executable:
$ go get github.com/dgrijalva/jwt-go/cmd/jwt


## 3- Add jwt to path:
$ PATH=$PATH:~/go/bin


# Steps to authenticate access to a virtual machine noVNC console

## 1- Create the following variables with virtual machine name, namespace, path, token expiry in seconds, URL for oc-gate route:
``` bash
$ vm=rhel6-150.ocp4.xxx.xxx (Replace with VM name)
$ ns=ocs-cnv (Replace with namespace where VM resides)
$ path=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/$ns/virtualmachineinstances/$vm/vnc
$ token_expiry=3600 (Replace with desired token duration in seconds)
$ keyfile=/home/aadib/console-access/test/key.pem (Replace with location of where the SSL key was created)
$ ocgateroute="oc-gate.apps.ocp4.xxx.xxx" (Replace with correct route path)
```

## 2- Set and display POST service URL:
$ posturl=https://$ocgateroute/login.html

$ echo $posturl
``` bash
https://oc-gate.apps.ocp4.xxx.xxx/login.html
```

## 3- Create and diplay JWT token signed by private SSL key:
$ TOKEN=$(echo {\\"exp\\": $(expr $(date +%s) + $token_expiry),\\"matchPath\\":\\"^/$path\\"} | jwt -key $keyfile -alg RS256 -sign -)

$ echo $TOKEN
``` bash
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhbGxvd2VkQVBJUmVnZXhwIjoiXi9wYXRoIiwiZXhwIjoxNjE0ODk1NjAwfQ.j6AqKritRobMWoKjUGjnp7Khntxsr2BsXZ2-GZmb20VLBAX4r6VDzsN4VP5wBalDjYn8o0mlt7kJ4BWy81hMOLWst8TD-d3Vt6xXr0Eo8rVUnodjXP_YctO4lHT1eoizNFnook80XTsHoDgXEGm04nqoKbIB71Re-7cQFZQSfWFPjUM4Qbl32ebFqfjDI-29UoerB3M5eyonYhmLHLS9LlL_XRbaDh1XOBEDMwQ9jQMw5fLQ2P7wtmyVHkHkUqmaA9d51KKuiGQrz0mQtdiHaq_DQYkoZ9Z47eZHrlOUlcAS7IEfaw3ZSCLB9kwXExQ5X0BmYP7hqvHeQTPsd1aWVg
```

## 4- Set and display POST path:
$ postpath=/noVNC/vnc_lite.html?path=$path

$ echo $postpath
``` bash
/noVNC/vnc_lite.html?path=k8s/apis/subresources.kubevirt.io/v1alpha3/namespaces/ocs-cnv/virtualmachineinstances/rhel6-150.ocp4.xxx.xxx/vnc
```

## 5- Open a web browser and enter post service URL from step 2:
![Screenshot from 2021-03-08 13-12-17](https://user-images.githubusercontent.com/77073889/110363740-eb460a00-8010-11eb-8e7a-256a6c42302c.png)


## 6- Enter token from step 3 in token field and the POST path from step 4 in the Then field, then click Submit:
![Screenshot from 2021-03-08 13-29-39](https://user-images.githubusercontent.com/77073889/110364968-6eb42b00-8012-11eb-92f0-cabe751ec733.png)


## 7- Press enter a couple of times and you will have access to the console:
![Screenshot from 2021-03-08 13-32-08](https://user-images.githubusercontent.com/77073889/110365266-d4a0b280-8012-11eb-8a89-26bd1d58be21.png)
