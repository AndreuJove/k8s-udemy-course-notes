Resources:

https://github.com/ricardoandre97/k8s-resources

---

# Pods:
Contenedor en Docker(aislados unos de los otros):
- IPC: inter process communication
- Cgroup
- Network (cada contenedor tiene una red). La comunicación entre ellos a partir de Bridge.
- Mount
- PID
- User
- UTS - Unix Timesharing System (hostname)

---

Pod en Kubernetes:
Definición: 1 o más contenedores que comparten namespace entre si.
- Tiene 1 IP para acceder al pod.
Sólo comparten:
- IPC
- Network
- UTS (hostname)



# minikube
Error networking localhost for minikube docker
$ minikube docker-env
Para la leccion Servicio de Tipo NodePort
```tex
Si usas el --driver=docker para iniciar minikube te dejo la siguiente nota y puedas ver que el NodePort si funciona.

  Localizar el puerto que te asigno el Services con el siguiente comando:
$ kubectl get services -l app=backend

Despues buscamos la IP de minikube para ver que nuestro servicio funciona:
$ minikube ip

Comprobar que esta funcional:
$ curl IP_minikube:port_service
```


---
# kubectl
```
in .bashrc add:
alias k='kubectl'
```
$k api-versions
$ k api-resources

# Pods:
$ k top pods
$ k run podtest --image=nginx:alpine
$ k describe pod podtest
$ k logs podtest
$ k get pod podtest -o yaml
$ k delete pod podtest
$ k port-forward podtest 7000:80
$ k cluster-info

$ k edit pod pod-test
$ k get pods --watch

$ k get pods -l app=backend


- Con YAML:
$ k apply -f pod.yaml


# ReplicaSet:
- Objeto encima de pod. A través de los labels hace réplicas de pods.
- Pone Owner Reference a los pods para saber quién es su ReplicaSet:
Ejemplo en k get pod rs-test-6s878 -o yaml
```yaml
ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: rs-test
    uid: 67bc607e-aee1-496b-b4ca-b875a157ba03
```

$ k get replicaset == k get rs
$ k label pods podtest app=pod-label


# Deployment:
$ k get deployment --show-labels
$ k rollout status deployment deployment-test
$ k annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"
$ k rollout history deployment deployment-test --revision=3
$ k rollout undo deployment deployment-test --to=revision=6

# Endpoints:
List of the IPs of the pods. Related to services
$ k get endpoints

# Service:
Permite comunicarse con los pods a través a través de labels. Las IPs de los pods pueden cambiar pero la del Service no. Usa Objecto Endpoint: una lista de Ips de los pods con el nombre del pod.

Types:
- ClusterIP: internal ip.
- NodePort: puerto que se abre a nivel del puerto.
- LoadBalancer.

$ k get svc
$ k get svc -l app=front-service
$ k describe svc my-service

To debug:
$ k run --rm -it pod-removed --image=nginx:alpine -- sh

$ k port-forward svc/backend-k8s-hands-on 7000:80

---

# Docker
$ docker run --rm -dti -v $PWD/:/go --net host --name golang-container golang bash


# Namespace:
Es un cluster virtual que permite separar distintos objectos de k8s isolados. Ejemplo: dev, prod.

- kube-public: can access everyone.
- kube-system: objects from kubernetes.

$ k get ns
$ k get all -n kube-system
$ k create namespace test-ns
$ k describe ns test-ns


- Access from another namespace:
k run --rm -it pod-removed --image=nginx:alpine -- sh
!!!
svcName + nsName + "svc.cluster.local"
!!!
Example:
curl backend-k8s-hands-on.ci.svc.cluster.local

### Context:
$ k config current-context
$ k config view
$ k config set-context ci-context --namespace=ci --cluster=minikube --user=minikube

$ k config use-context ci-context
Rollout:
k config use-context minikube
---


# Limits & Requests:
Requests: garantizado por k8s
Limits: no garantizado. Depende de como esté el nodo.

Para ver que capacidad tiene el nodo:
$ k get nodes
$ k describe node minikube

# QoS (Quality of Service) for Pods:
3 types:
- Guaranteed: limits == requests
- Burstable: limits > requests
- BestEffort: no limits

# LimitRange:
Configuraciones para namespaces a nivel de pod (individual). Puede generar problemas si hay muchas réplicas del pod. Necesita ResourceQuota.

k get limitrange -n dev


# ResourceQuota:
Actua a nivel de namespace.

# Probes:
Kubelet ejecuta a través de 
- command
- TCP: is port X running?
- HTTP al /path
sobre los pods para saber si estan corriendo.

Types in K8s:
**Liveness**: cada n tiempo se ejecuta en el pod
**Readness**: se ejecuta en el pod cuando se crea para verificar que funciona antes de ponerlo como endpoint.
**Startup**: se ejecuta antes del Liveness y el Readness. Si tarda mucho a encenter la app.


# Envs:




# Configmap:
Separar las configuraciones del pod, para que las pueda consumir. Se puede inyectar variables de entorno o directamente archivos a través de volumenes. 

Mirar carpeta configmaps

- Key:value (Always)

$ k create configmap nginx-config --from-file=configmaps-examples/nginx.conf
$ k get cm


# Secret:
Parecido a un configmap.


$ k create secret generic mysecret --from-file=./secret-files/test.txt
$ k describe secret mysecret
$ echo c2VjcmV0MT1ob2xhCnNlY3JldDI9YWRpb3MK | base64 --decode

Usando en el yaml data tienen que estar en base64.
Usando stringdata no hace falta que estén en base64


- 1 manera segura:
**envsubst**
export USER=andreu
export PASSWORD=paco
envsubst < secure.yaml > paco.yaml
---

# Volumenes:

- emptyDir: crea un directorio a nivel de pod. Los distintos contenedores pueden usar el EmptyDir. Si muere el pod muere el EmptyDir.
- hostPath: depende de la vida del nodo. Problema de si el pod va a otro nodo: se pierde la información.
- Cloud Volums: apuntas a EBS (AWS) o PD(GCP) hay que saber el id, el disco, etc. No pierde la data.

- PV(Persisten Volume) y PVC(Persisten Volume Claim):
PV es el que guarda el volumen. El PVC es una reclamación.
El PV conecta con AWS EBS por ejemplo
  Eliminar PVC que hacemos?
  Dependiendo de la reclaim policy:
    - Retain: el pv no se elimina ni la data
    - Recycle: el PV no se elmina pero la data si se elimina. Lo deja disponible para volver a reclamarse (PVC).
    - Delete: se borra el PV y la data


Storage class: si no hay pv y hay pvc se crea el pv de tipo hostpath default (esto se llama Provisinamiento Dinámico). Para verlo: 
$ k get sc



# RBAC (Rol-based access control):
Roles vs Cluster Roles:

Los Roles aplican solo a un namespace.
Los ClusterRoles aplican a todos los namespaces del clúster.

Como crear usuarios:
- Token
- Auth
- Certificates

CA (Certification Authority): certificado para firmar otros certificados. Data muy sensible tiene que estar muy escondida.

PASOS para generar certificado+signarlo por CA:
1) Generar certificado.
2) CSR -> generar archivo para la firma del CA
  - Campo CN (k8s lo usa como User)
  - Campo O (k8s lo usa como group)
3) Signar.


### Steps:
minikube start --vm-driver=none  --extra-config=apiserver.authorization-mode=RBAC

##### Create keys and sign
openssl genrsa -out ricardo.key 2048
openssl req -new -key ricardo.key -out ricardo.csr -subj "/CN=ricardo/O=dev"

>Para saber donde está el ca :
$ k config view

>Firmar con el ca:
sudo openssl x509 -req -in ricardo.csr -CA /root/.minikube/ca.crt -CAkey /root/.minikube/ca.key -CAcreateserial -out ricardo.crt -days 500
openssl x509 -in  ricardo.crt  -noout -text

## Isolated env
kubectl config view  | grep server
docker run --rm -ti -v $PWD:/test -w /test  -v /root/.minikube/ca.crt:/ca.crt -v /usr/bin/kubectl:/usr/bin/kubectl alpine sh

## Configure kubectl for user
kubectl config set-cluster minikube --server=https://192.168.1.140:8443 --certificate-authority=/ca.crt
kubectl config set-credentials ricardo --client-certificate=ricardo.crt --client-key=ricardo.key
kubectl config set-context ricardo --cluster=minikube --user=ricardo
kubectl config use-context ricardo

k cluster-info dump

RBAC verbs:
https://kubernetes.io/docs/reference/access-authn-authz/authorization/#determine-the-request-verb

# ClusterRole:
k get clusterroles


# RoleBinding:
Permite unir un Rol a un User.




# Service Account:
Es como un Rol. Tiene asociado un token. Sirve para comunicación entre pods. Siempre se crea un service account por namespace.

$ k get sa
$ k get sa -n kube-system


Para atacar la api de k8s usar el JWT


# Ingress:
Exponer servicios fuera del clúster. A través de path redirige a distintos services.
**Ingress controller**: es el que aplica las reglas del Ingress definidas en el yaml

k get ingress


# EKS:
Amazon gestiona el nodo de Master.

$ aws sts get-caller-identity

**ekctl**

$ eksctl create cluster --name test-cluster --without-nodegroup --region us-east-1 --zones us-east-1b, us-east-1a


- Crear de nuevo el .kube/config:
aws eks --region us-east-1 update-kubeconfig --name test-cluster

$ k cluster-info


- Crear nodos:

eksctl create nodegroup \
  --cluster test-cluster \
  --name al-nodes \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --ssh-access \
  --managed=false \
  --ssh-public-key my-key


**Ingress en EKS**

https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

Pasos:
1. Crear Service Account. 
  - Necesita un IAM Role para él para crear ALB

2. Desplegar Ingress Controler que usa el service account que hemos creado


# HPA (Horizontal Pod Autoscaler):
Consulta fuente de métricas para crear o bajar el número de pods.

Manera de hacerlo a través de terminal (es mejor hacerlo en manifiesto con el kind: HorizontalPodAutoscaler)
$ k autoscale deployment httpd --cpu-percent=50 --min=1 --max=10 

$ k get hpa


# Cluster AutoScaler:

https://docs.aws.amazon.com/es_es/eks/latest/userguide/autoscaling.html