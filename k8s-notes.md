general concepts

storage %10
storageClass, pv , pvc
volumeMode, accessMode, reclaimPolicy  for volumes
pvc primitive
- how to configure apps that needs persistent storage

troubleshooting *** %30

https://devopstitan.com/ckubernetes-practice-questions/
https://kubernetes-tutorial.schoolofdevops.com/vote-deployement_strategies/#case-for-service-mesh
https://rudimartinsen.com/2021/01/14/cka-notes-troubleshooting/

cluster architecture , installation , configuration  ** % 25

	RBAC
	using kubeadm
	manage HA cluster
	provision infras. to deploy a cluster 
	version upgrade
	etcd backup and restore * 

workload - scheduling % 15
understand deployment,  rolling update & rollback
use configMap and secrets and environment
scale applications
understand primitives
resource limits for pod scheduling
manifest management and templating tools


services & networking % 20
host networking configuration
connectivity btw nodes
ClusterIp , NodePort, LoadBalancer **
configuring and using coreDNS
choose container-network-interface-plugin

	understanding kubernetes architecture - comprehensive
https://devopscube.com/kubernetes-architecture-explained/




install kubernetes from scratch with kubeadm :

kaynak ;
https://devopscube.com/setup-kubernetes-cluster-kubeadm/


master node components :
etcd
scheduler
kube-apiserver
controller manager
cloud-control-manager

worker node components :
container runtime (docker,containerd, cri-o , etc.)
kubelet
kube-proxy





practise jq commands .

etcd notes:

kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"




use KODEKLOUD20 code when registering the cka exam to got %20 discount.
install k8s on your local machine ;
resource : https://thechief.io/c/editorial/k3d-vs-k3s-vs-kind-vs-microk8s-vs-minikube/





imperative commands:

alias k=kubectl
export do="--dry-run=client -o yaml"

--genel
kubectl cluster-info
kubectl get nodes
kubectl api-resources

-- namespace
kubectl create ns erkan
To switch another namespace permanently.
kubectl config set-context $(kubectl config current-context) --namespace=dev
kubectl get pods --all-namespaces
kubectl get pods -n egys
--resource quota
(To limit resources in a namespace, create a resource quota)
apiVersion: v1
kind: ResourceQuota
metadata:
name: compute-quota
namespace: dev
spec:
hard:
pods: "10"
requests.cpu: "4"
requests.memory: 5Gi
limits.cpu: "10"
limits.memory: "10Gi
-- pod
kubectl run nginx --image=nginx:alpine --port=8080 --env="ERKAN=123" --env="ONAT=234"
--labels=tier=db --restart=Never --expose=true  
delete pod without waiting : kubectl delete pod <name> --force
kubectl delete pod nginx --grace-period=0 --force
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600"
kubectl run busybox --image=nginx --restart=Never -it -- echo "How are you"


get yaml for running pod : kubectl get pod <name> -o yaml > pod.yaml
generic :
kubectl -n <namespace> get <resource type> <resource Name> -o yaml
--replicationController
equality based selector used.
spec. selector:
app: nginx
tier: dev
-- replicaset
set based selector used.
spec.selector:
matchLabels:
env: prod
matchExpressions:
- { key: tier, operator: In, values: [frontend] }


-- deployment
kubectl create deployment nginx --image=nginx:alpine
kubectl scale deployment webapp --replicas=3
kubectl edit deployment my-deployment
--rollout
kubectl set image deployment <name> nginx=nginx:1.1.17 --record
kubectl rollout history deployment <name>


-- service
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml

(servicename.namespace.service.domainname)
-- secret
kubectl create secret generic db-secret-xxdf --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
-- job
kubectl create job hello --image=busybox -- echo "Hello World"

-- persistent volume
apiVersion: v1
kind: PersistentVolume
metadata:
name: pv-analytics
spec:
capacity:
storage: 100Mi
volumeMode: Filesystem
accessModes:
- ReadWriteMany
  hostPath:
  path: /pv/data-analytics

-- pvc

-- network

-- authorization
kubectl config view

--accounts
-user account
-service account
kubectl create serviceaccount <account name>
kubectl get serviceaccount
kubectl describe serviceaccount <account name>

When a service account is created, it also creates a token automatically

for authentication to private registry
create secret of type docker-registry and set imagepullsecret type of that secret or do patch as follow:
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'

at the end:
--dry-run=client -o yaml > <filename>.yaml

listing pods with specific labels ;

kubectl get pods --selector=name=nginx,type=frontend

Editing pod with 3 alternative:

1- kubectl edit pod <pod-name>
kubectl delete pod <name>
kubectl create -f /tmp/temp-updated-pod.yml

2- kubectl get pod <pod-name> -o yaml  > my-new-pod.yaml
vi my-new-pod.yaml
kubectl delete pod <existing-pod>
kubectl create -f my-new-pod.yml

3- kubectl edit deployment my-deployment.


create pod:
k run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
k create -f pod.yaml
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: nginx
name: nginx
spec:
containers:
- image: nginx
  name: nginx
  ports:
  - containerPort: 80
    command: ["sleep"]
    args: ["10"]
    readinessProbe:
    httpGet:
    path: /api/ready
    port: 8080
    resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Never
    status: {}


create deployment:
kubectl create deployment <name> --image=nginx:latest --dry-run=client -o yaml > dep.yaml
update deployment:
kubectl rollout status deployment/<name>
kubectl get deployments

updating deployment:
kubectl set image deployment/<dep-name>  --record

rollout and rollback :
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout history deployment/nginx-deployment --revision=2
rollback :
kubectl rollout undo deployment/nginx-deployment
rollback to specific version
kubectl rollout undo deployment/nginx-deployment --to-revision=2

k annotate sts tb-node kubernetes.io/change-cause="blabla"

scale deployment:
kubectl scale deployment/<name> --replicas=3
kubectl autoscale deployment/<name> --min=2 --max=5 --cpu-percent=80

example yaml :


readinessProbe and livenessProbe:
example:
livenessProbe:          # add
tcpSocket:           # add
port: 80           # add
initialDelaySeconds: 10    # add
periodSeconds: 15       # add

temelde 3 şekilde readiness durumu kontrol edilebilir.
spec.containers[0].readinessProbe:
httpGet:
path: /api/ready
port: 8080
tcpSocket:
port:  3306
exec:
command:
-cat
- /app/iss_ready



örnek.
C


taint and tolerations:

label & selector & annotation:


create or expose service :
service types:
clusterIP
NodePort
Headless
create a service:

create service :  kubectl create service <type> <name> --tcp=80:80 --dry-run=client
expose pod : kubectl expose pod redis --port=6379 --name=redis-svc --type=NodePort

replication-controller:
example yaml:

replica Sets:
example yaml :

statefulSet:
example yaml :

Jobs :
example yaml :

CronJobs
example yaml :

ConfigMap:
kubectl create configmap <name> --from-literal=<key>=<value>

Secrets
kubectl create secret generic <name> --from-literal=<key>=<value>
kubectl create secret generic <name> --from-file=<path-to-file>

encode a secret:
echo -n ‘blabla’ | base64

decode a secret:
echo -n ‘xyzvadf==’ | base64 --decode

inject into pod :
spec.containers[0].
envFrom:
- secretRef:
name: secret-name

    env:
      -name: db_password
       valueFrom:
         secretKeyRef:
           name: app-secret
           key:  db_password

volumes:
- name: app-secret-volume
secret:
secretName: app-secret



Volume:
example yaml :
apiVersion: v1
kind: Pod
metadata:
name: test-pd
spec:
containers:
- image: k8s.gcr.io/test-webserver
  name: test-container
  volumeMounts:
  - mountPath: /test-pd
    name: test-volume
    volumes:
- name: test-volume
  hostPath:
  # directory location on host
  path: /data
  # this field is optional
  type: Directory















PersistentVolume:
example yaml :
apiVersion: v1
kind: PersistentVolume
metadata:
name: pv-log
spec:
persistentVolumeReclaimPolicy: Retain
accessModes:
- ReadWriteMany
capacity:
storage: 100Mi
hostPath:
path: /pv/log


PVC: (persistent-volume-claim)
example yaml :

storage classes
example yaml :

network policy:
example yaml :

create pod with single or multiple containers
update pod environment variable

create namespace:
kubectl create namespace <name>

connect on a service which listens different ns:
mysql.connect(“<name-of-service>.namespace.svc.cluster.local”);

change current namespace:
kubectl config set-context $(kubectl config current-context) --namespace=dev

get pods in all ns:
kubectl get pods --all-namespaces
kubectl get pods -n <namespace>

create deployment
create replica set
create service
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
kubectl expose deployment redis -n marketing --type=ClusterIP --name=messaging-service --port=6379 --dry-run=client -o yaml > svc.yaml

create configmap
create secret file


extra concepts :
deployment strategies:
-> blue green
-> canary


ingress:
kubectl create ingress <ingress-name> --rule="host/path=service:port
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: minimal-ingress
annotations:
nginx.ingress.kubernetes.io/rewrite-target: /
spec:
ingressClassName: nginx-example
rules:
- http:
  paths:
  - path: /testpath
    pathType: Prefix
    backend:
    service:
    name: test
    port:
    number: 80


network policies:



docker commands:

Dockerfile format = instruction-argument
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]


docker image ls
docker build Dockerfile -t  erkan/myap1:latest
docker push <registry-host:port>/myname/myimage
docker run -it -p 5434:5432 --name <container-name> myap1

--
path :   $HOME/.kube/config
kubectl config view
kubectl config use-context user@cluster
kubectl config -h
~/.kube/config  file

apiVersion: v1
kind: Config
currentContext : user@cluster

clusters:
-
-
contexts:
-
-
users:
-
-

kubectl config --kubeconfig=/root/my-kube-config use-context research
kubectl config --kubeconfig=/root/my-kube-config use-context current-context
default location of kubeconfig file : $HOME/.kube/config


kubectl convert:

kubectl convert -f <old-file> --output-version apps/v1
upgrades the apiVersion field of the yaml.


QoS:
Guaranteed
Burstable
BestEffort
spec:
containers:
...
resources:
limits:
cpu: 700m
memory: 200Mi
requests:
cpu: 700m
memory: 200Mi
...
status:
qosClass: Guaranteed

# List the environment variables defined on a deployments 'sample-build'
kubectl set env deployment/sample-build --list

You can use kubectl replace -f to update a live object according to a configuration file.


kubectl patch examples:




game of pods:
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: drupal
app: drupal
name: drupal
spec:
replicas: 3
containers:
- name: init-sites-volume
  image: drupal:8.6
  command: ["/bin/bash", "-c", "cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R"]
  volumeMounts:
  - name: myvolume
    mountPath: "/data"
- image: drupal:8.6
  name: drupal
  volumeMount:
  - name: modules
    mountPath: "/var/www/html/modules"
    subPath: modules
  - name: profiles
    mountPath: "/var/www/html/profiles"
    subPath: profiles
  - name: sites
    mountPath: "/var/www/html/sites"
    subPath: sites
  - name: themes
    mountPath: "/var/www/html/themes"
    subPath: themes
    resources: {}
    volumes:
- name: myvolume
  persistentVolumeClaim:
  - claimName: drupal-pvc
    dnsPolicy: ClusterFirst
    restartPolicy: Always
    status: {}




--
apiVersion: v1
kind: PersistentVolume
metadata:
name: drupal-pv
spec:
capacity:
storage: 5Gi
volumeMode: Filesystem
accessModes:
- ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
  path: /drupal-data
  nodeAffinity:
  required:
  nodeSelectorTerms:
  - matchExpressions:
    - key: app
      operator: In
      values:
      - drupal

--nodeAffinity example:
pod.spec:
nodeSelector:
disktype: ssd
or:
affinity:
nodeAffinity:
requiredDuringSchedulingIgnoredDuringExecution:
nodeSelectorTerms:
- matchExpressions:
- key: kubernetes.io/e2e-az-name
operator: In
values:
- e2e-az1
- e2e-az2


apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx
spec:
affinity:
nodeAffinity:
preferredDuringSchedulingIgnoredDuringExecution:
- weight: 1
preference:
matchExpressions:
- key: type
operator: In
values:
- t2medium
requiredDuringSchedulingIgnoredDuringExecution:
matchExpressions:
- key: type
operator: In
values:
- t2medium
containers:
- name: nginx
  image: nginx

---
apiVersion: apps/v1
kind: Deployment
metadata:
creationTimestamp: null
labels:
app: drupal-mysql
name: drupal-mysql
spec:
replicas: 1
selector:
matchLabels:
app: drupal-mysql
strategy: {}
template:
metadata:
creationTimestamp: null
labels:
app: drupal-mysql
spec:
containers:
- image: mysql:5.7
name: mysql
resources: {}
volumeMounts:
- name: mysql-volume-claim
mountPath: /var/lib/mysql
subPath: dbdata
volumes:
- name: mysql-volume-claim
persistentVolumeClaim:
claimName: drupal-mysql-pvc
status: {}

--
apiVersion: v1
kind: PersistentVolume
metadata:
name: drupal-pv
spec:
capacity:
storage: 5Gi
accessModes:
- ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
  path: /drupal-data


tips:
alias k=kubectl

<set default namespace>
$ kubectl config set-context <context-of-question> --namespace=<namespace-of-question>

delete resources quickly :
kubectl delete pod nginx --grace-period=0 --force

kubectl get pods -o yaml | grep -C 5 labels:
kubectl describe pods | grep -C 10 "author=John Doe"

important tips !!!
--
simulator answers :

alias k=kubectl                         # will already be pre-configured
export do="--dry-run=client -o yaml"    # k get pod x $do
export now="--force --grace-period 0"   # k delete pod x $now
alias kn='kubectl config set-context --current --namespace '
kn default        # set default to default
kn my-namespace   # set default to my-namespace

---  vim ~/.vimrc
set tabstop=2
set expandtab
set shiftwidth=2
---



install k3s on macos : https://andreipope.github.io/tutorials/create-a-cluster-with-multipass-and-k3s.html
install kubectl_aliases :
curl https://raw.githubusercontent.com/ahmetb/kubectl-aliases/master/.kubectl_aliases --output /home/erkan/.kubectl_aliases

give environment variable to spring boot app from k8s :
https://www.tutorialworks.com/spring-boot-kubernetes-override-properties/

--installing helm ;
https://helm.sh/docs/intro/install/
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh

--deploy kafka & zookeeper on k8s cluster with bitnami/helm ;
https://docs.bitnami.com/tutorials/deploy-scalable-kafka-zookeeper-cluster-kubernetes/

port requirements :

https://github.com/techiescamp/kubeadm-scripts

Install container runtime on all nodes- We will be using cri-o.
Install Kubeadm, Kubelet, and kubectl on all the nodes.
Initiate Kubeadm control plane configuration on the master node.
Save the node join command with the token.
Install the Calico network plugin (operator).
Join the worker node to the master node (control plane) using the join command.
Validate all cluster components and nodes.
Install Kubernetes Metrics Server
Deploy a sample app and validate the app












