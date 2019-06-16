
1) Create Pod with label app=run

$ kubectl run nginx --image=nginx --restart=Never -l=app=run --dry-run -o yaml

2) Create a Pod with label app=run and command /bin/bash

$ kubectl run nginx --image=nginx --restart=Never -l=app=run --dry-run -o yaml --command -- /bin/sh 

3) Create a deployment with environment variable test1=true

$ kubectl run nginx --image=nginx  -l=app=run --env=test1=true --dry-run -o yaml --command -- /bin/sh 

4) Create Deployment with container port = 80

$ kubectl run nginx --image=nginx  -l=app=run --env=test1=true  --port=80 --dry-run -o yaml 

5) Create a multi label deployment

$ kubectl run nginx --image=nginx  -l='app=run,run=fast' --env=test1=true  --port=80 --dry-run -o yaml 

6) Deployment with 5 replicas

$ kubectl run nginx --image=nginx --replicas=5 --dry-run -o yaml

7) Create a Job name pi with label app=run with args bash

$ kubectl run pi --image=perl --restart=OnFailure -l=app=run --dry-run -o yaml -- bash

8) Create a schedule job 

$ kubectl run pi --schedule="0/5 * * * ?" --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'

9) Create a deployment with hardlimit (limit)  QoS Class: Guaranteed

$ kubectl run nginx --image=nginx --replicas=5 --limits='cpu=100m,memory=256Mi' --dry-run -o yaml

10) Create a deployment with softlimit (requests)   QoS Class: Burstable

$ kubectl run nginx --image=nginx --replicas=5 --requests='cpu=100m,memory=256Mi' --dry-run -o yaml

11) Create Pod with serviceaccount "acc1"

$ kubectl run nginx --image=nginx --limits='cpu=100m,memory=256Mi' --serviceaccount=acc1 --dry-run -o yaml

12) Create a Cron job (scheduled)

$ kubectl run perl -l=perljob=true --schedule="0/5 * * * ?" --image=perl --restart=OnFailure --dry-run -o yaml

13) Switch context

$ kubectl config use-context <Context-Name>


================================================================================================

$ kubectl expose

Expose a resource as a new Kubernetes service. 

Looks up a deployment, service, replica set, replication controller or pod by name and uses the selector for that
resource as the selector for a new service on the specified port. A deployment or replica set will be exposed as a
service only if its selector is convertible to a selector that service supports, i.e. when the selector contains only
the matchLabels component. Note that if no port is specified via --port and the exposed resource has multiple ports, all
will be re-used by the new service. Also if no labels are specified, the new service will re-use the labels from the
resource it exposes. 

Possible resources include (case insensitive): 

pod (po), service (svc), replicationcontroller (rc), deployment (deploy), replicaset (rs)


1) Create a service for a replicated nginx, which serves on port 80 and connects to the containers on port 8000.

$  kubectl expose deployment nginx --port=80 --target-port=8000 --dry-run -o yaml

2) Expose container with service name as http-nginx 

$ kubectl expose pod nginx --name http-nginx --port=80 --target-port=80 --dry-run -o yaml

3) Create a service with Loadbalanced type --type='': Type for this service: ClusterIP, NodePort, LoadBalancer, or ExternalName. Default is 'ClusterIP'.

$ kubectl expose pod nginx --name http-nginx --port=80 --target-port=80 --type=LoadBalancer --dry-run -o yaml

4) Create Headless service (stateful sets)

$ kubectl expose pod nginx --name http-nginx --port=80 --target-port=80 --type=ClusterIP --cluster-ip='None' --dry-run -o yaml

5) Create a service for a replicated streaming application on port 4100 balancing UDP traffic and named 'video-stream'.

$ kubectl expose rc streamer --port=4100 --protocol=udp --name=video-stream

6) Create stateful sets

$ kubectl run dep1 --image=nginx -l=app=run --dry-run -o yaml > /tmp/1.txt

$ echo "---" >> /tmp/1.txt 

$ kubectl expose deployment nginx --port=80  --target-port=80 --type=ClusterIP --cluster-ip=None --dry-run -o yaml >> /tmp/1.txt 

modify:
   apiVersion: apps/v1
   kind: StatefulSet
   serviceName: <service name>

=================================================



1) Create Service account "sa1"

$ kubectl create serviceaccount sa1

2) Create secret se1 from literal

$ kubectl create secret generic se1 --from-literal=abc=1

3) Create a new secret named my-secret with key1=supersecret and key2=topsecret

$ kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret

4) Create a new secret named my-secret from an env file

$ kubectl create secret generic se2 --from-env-file=/tmp/secret

$ cat /tmp/secret 
test1=1
test2=2

5) Create secret file to mount as volume

$ kubectl create secret generic se3 --from-file=/tmp/secret 

6) Mount secrets as a file into container

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx1
  name: nginx1
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: busybox
    volumeMounts:
    - name: vol1
      mountPath: /tmp/1
  volumes:
  - name: vol1
    secret:
      secretName: se3
  dnsPolicy: ClusterFirst
  restartPolicy: Never

7) Create configmap 

$ kubectl create configmap conf1 --from-file=/tmp/secret

8) Map secret to the deployment as environment variables.

$ kubectl set env deployment nginx --from=secret/se3

9) Map Configmap to the deployment as environment variables.

$ kubectl set env deployment nginx --from=configmap/conf1

10) Listen on port 8888 locally, forwarding to 5000 in the pod

$ kubectl port-forward pod/mypod 8888:5000

11) Create a new namespace named my-namespace

$ kubectl create namespace my-namespace

12) Create role foo for services,pods and secrets

$ kubectl create role foo --verb=get,list,watch --resource=pods,secrets,services --dry-run -o yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: foo
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - secrets
  - services
  verbs:
  - get
  - list
  - watch

13) Create rolebinding

$ kubectl create rolebinding foobinding --role=foo --serviceaccount=default:sa1 --dry-run -o yaml

rolebinding.rbac.authorization.k8s.io "foobinding" created

14) Set subject in the role binding

$ kubectl set subject rolebinding foobinding --serviceaccount=default:sa1 --dry-run -o yaml

15) Create a new resourcequota named my-quota

$ kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10,limits.memory=200Mi
  
16) Copy /tmp/foo local file to /tmp/bar in a remote pod in namespace <some-namespace>
   
$ kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar

17)Copy /tmp/foo_dir local directory to /tmp/bar_dir in a remote pod in the default namespace

$ kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir

18)Set Deployment nginx-deployment's ServiceAccount to serviceaccount1

$ kubectl set serviceaccount deployment nginx-deployment serviceaccount1
  
