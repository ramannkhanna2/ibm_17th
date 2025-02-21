# ibm_17th





```

Welcome to OpenDev Etherpad!


https://github.com/ramannkhanna2/ibm_17th.git

kubernetes : manager which managed the containers    : container orchestrater  : docker swarm , mesos , etc 


docker : engineer (container runtime ) podman, rocketd , etc 
    
    docker >>> create containers 
    
    container (light weight ) as compared vms 
    
    
    dockerfile >>> docker build >>   docker image of ur application  (base images) >>>>   container  applications 
    
    --------  docker run --name c1 redis -P
    
    -----   docker run 
    
 ==========================================================================================
 
 
 creating my cluster :
     
     pre requisite : 1 master , 2 workers : 3 machines ( aws , azure , gcp , any machine)
     
     
     
       User name         ibm_17th
       
         Password        aK8745=^                      
         
           Console sign-in URL          https://685421549691.signin.aws.amazon.com/console             
     
     
     region : ohio ...........
     
     ==============
     
     
     raman-kube-
     
     3 machines ..
     
     ami : ubuntu 
     
     instance-type : t2.medium 
     
     create a keypair raman-ibm-key
     
     reuse my sg / create a new one : raman-sg
     
     storage : 20 gb
     
     ===============================================
     
     
     ubuntu@ip-172-31-0-116:~$ sudo -i
root@ip-172-31-0-116:~# hostnamectl set-hostname master
root@ip-172-31-0-116:~# bash
root@master:~#





ubuntu@ip-172-31-4-133:~$ sudo -i
root@ip-172-31-4-133:~# hostnamectl set-hostname w1
root@ip-172-31-4-133:~# bash
root@w1:~#


===================================================================

Kubernetes installation steps:


1. create a script on each machine

vi script.sh


```

#!/bin/bash

# Update package lists and install Docker
sudo apt update -y
sudo apt install docker.io -y

# Set up Kubernetes repository and install kubelet, kubeadm, and kubectl
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Enable net.bridge.bridge-nf-call-iptables
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# Download and install cri-dockerd
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.14/cri-dockerd-0.3.14.amd64.tgz
tar -xvf cri-dockerd-0.3.14.amd64.tgz
cd cri-dockerd
sudo install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd

# Download and set up cri-dockerd systemd service
cd ..
wget https://github.com/Mirantis/cri-dockerd/archive/refs/tags/v0.3.14.tar.gz
tar -xvf v0.3.14.tar.gz
cd cri-dockerd-0.3.14/
sudo cp packaging/systemd/* /etc/systemd/system
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

# Enable and start cri-docker service
sudo systemctl daemon-reload
sudo systemctl enable --now cri-docker.socket
sudo systemctl enable cri-docker
sudo systemctl start cri-docker
sudo systemctl status cri-docker


```

2. create a config file on the master machine and mention the master node private ip address (advertiseAddress)

vi config.yaml

apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 172.31.3.
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
networking:
  podSubnet: 192.168.0.0/16


kubeadm init --config=config.yaml >> cluster_initialized.txt



3. cat cluster_initialized.txt     # run below commands   ( on master )


mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config



4. install the calico plugin on master node

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

5. join the worker node using --cri-socket unix:///var/run/cri-dockerd.sock

kubeadm join 172.31.3.203:6443 --token 7gy0n6.gycf0eed7x3lihdj \
        --discovery-token-ca-cert-hash sha256:6dc79151d4cd2c5e1e05e912f7935cf4cae1df149a2f3a74a62d84ee8bbe3cbf \
        --cri-socket unix:///var/run/cri-dockerd.sock

        
        ==========================================================================================
        
        
        history :
            
            
            kubectl get nodes
   42  kubectl get pods
   43  kubectl get pods -A
   44  kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
   45  kubectl get nodes
   46  kubectl get pods -A
   47  clear
   48  kubectl get nodes
   49  kubectl get pods -A
   50  clear
   51  kubectl get pods -A
   52  kubectl get pods -A -o wide
   53  kubectl get nodes
   54  ls
   55  cat cluster_initialized.txt
   56  kubectl get nodes
   57  clear
   58  kubectl get nodes
   59  kubectl get pods -A
   60  clear
   61  kubectl get pods -A -o wide


============================================




 kubectl                                                                     to create a container /set of containers (pods)  and manage them using kubernetes api




 kubectl get pods
   82  clear
   83  kubectl get nodes
   84  kubectl run ramanapp --image httpd
   85  kubectl get pods
   86  kubectl get pods -A
   87  clear
   88  kubectl get pods
   89  kubectl get pods -A
   90  kubectl get pods -A -o wide
   91  clear
   92  kubectl get pods
   93  kubectl get pods -n kube-system
   94  kubectl get pods -A
   95  kubectl get pods -o wide
   96  clear
   97  kubectl get pods
   98  kubectl get pods -A | grep -i master
   99  kubectl get pods -A | grep -i worker
  100  kubectl get pods -A | grep -i w1
  101  kubectl get pods -o wide
  102  clear
  103  kubectl get pods
  104  kubectl get pods -o wide
  105  kubectl delete po ramanapp
  106  kubectl get pods

==========================================================





-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAsdFGugVbhz4sWzpQ001G30EIIAtyDCE2raEd8ieVt9ixIbvj
kwvRp3UojtvmBlGs0vMOTsnFHtiYQZVNKb5HCxC9xlpw+jRgCBkkzDKyUdmZWHjq
1LqDijLWTcJSOdaQDsC5WxdRAUZn4jpiimRnvFDE1eyN/Auva11AubW0BUqG79vO
HZCdWJB2k5ALNCq6vbjZyk99ZI4mU9UmzC+4MF8uAR3O48dIda4oAF4cfRgJaFdY
0B62+D6BFy2stgSKJcwE7NBKVvW4xDIvc1N1Pwp279UZTC9qghwF+wOyPKxPyUEJ
qYRyh4tfDTjeriwzZO9NsRVZZ2CffvMI3Ca7DwIDAQABAoIBAQCZ4wBviqVshL0E
cpJyW6VjHre2a9FWeAQG/bGZ2PI0Oh8Jj75iis71OmpQQDRw8Yw8v8Z5HxsuF8qk
r14pKxf2lpV2LN0rW9pkB8aVxaYXOdcA/xxT39po5pgakXpxvaPMcLO5BpO/I7xR
x69yD3TLP6cpb+Bs6Xv10a1rSrox/Jyn56WLpOFQ4K/FtDuUVcHpEs58IPpEWgUU
rl115AY6H4ugRLm9myzvmqghqI6CTIEtMDSFKFmvaf/kmXLgjgIhzjhMyp3R1XEb
ZXoQI6PaIRQfmAxDqqZdV8MSwAxmLofuJ55AVDQ1QpST94DYhs3t/rwKmjmT7m//
sBugP2MBAoGBAOHjCW1uZrjOJcU6OpWRdsVbHrtf9yQ1bdO8N7wll2XfrpmCYVUU
16yBreC7RICKcwFB15R3rnBjjrFbVrpehWLFgPGPpp+pIMNigPBQzo8EwSvHbTvZ
8L1vmIfwOTuoXAZbEcK2nL9F59AEfeiFAM0/1/v+O89Kv5Vuqx8T/Z+PAoGBAMmF
v/NyyGTRwmKjbNfojxurrLHFvoNFs8I3QqDvZK40Atnst+0QBeP8vce43NrXbWdT
iyENz4hAM7CGFKaDPSb9ItYJfUP7ktaoczWIE8vv0Ow5Veo4nXZagEbaAcXME06d
DUPaTJdVO8p56x94MP1Fe4NAmW+dturPg6Uw52yBAoGBAMm9Pjc4yY81ta/+wEHC
h6PdLIZGP/Bbs3nN+K0VmbCHZGV/dzRIiBJuQv+Z4KU4gVvXFRVpCicgE1m87KlI
L7K2F+Il3LdtknBNskBuuvwqT+eslZdFnudhGoYV+teYFpAql6Mh+r4tTcqPqG+Z
Ec09vsU1Gu+Yn6BzFWuNLA9rAoGAQPIi75GBdcSIMgPbMyYW4OMN7+j7whC3oxLu
HTGpr97BQHxitjrgux4cB37TZo/hCVjKUOfDh3Sxc+VySEupbKROEs7SGRO9ugJl
xs3JG4N5QHgl8Ss3zAnUp4Dg618epcpFmSWEY40rjNCH7wdsOmOnL6ClEmywo7In
ChjdrIECgYAWkXYDwO0glvozLzDuCpk3nKNUM/nupTofbrZeS/5dBf0Z5SOOpfrs
VYDw726+be4VVK2K442REjDD8eUDXZtBa8ieta81/VNapE6rrf6WnCZdex8GbnZ/
8sLu4jlFm//gxVT9QO3zBTb5dIwhXug3/teWkxm6aGVYvecthevEDg==
-----END RSA PRIVATE KEY-----




========================================================================


master :
    
ssh -i "ibm-key.pem" ubuntu@ec2-18-221-128-77.us-east-2.compute.amazonaws.com

w1 :

ssh -i "ibm-key.pem" ubuntu@ec2-3-145-81-134.us-east-2.compute.amazonaws.com


w2 :
    
ssh -i "ibm-key.pem" ubuntu@ec2-18-191-238-82.us-east-2.compute.amazonaws.com

============================================================================
    
    --- create a namespace :
    
 kubectl get nodes
  116  alias k=kubectl
  117  k get po
  118  k get pods -A
  119  k get ns
  120  k create ns raman
  121  clear
  122  k get ns
  123  k run ramanapp1 --image httpd -n raman
  124  k get pods
  125  k get pods -n raman
  126  k get pods -A
  127  k get pods -n raman -o wide

==============================



commandline way , decalaraative







 k get ns
  130  clear
  131  kubectl api-resources
  132  clear
  133  k api-resources
  134  ls
  135  vi pod.yaml
  
  
  
apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
spec:
  containers:
  - name: nginxcon
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    
    
    
  136  cat pod.yaml
  137  k create -f pod.yaml -n raman
  138  k get pods
  139  k get pods -n raman
  140  k delete pods --all -n raman
  141  ls
  142  cat pod.yaml
  143  vi pod.yaml
  144  k create -f pod.yaml
  145  k get pods -n raman


==========================================================


 k get pods -A
  149  k get pods -A -o wide
  150  clear
  151  k get pods
  152  k get pods -n raman
  153  k get pods -n raman -o wide
  154  k describe po nginxpod -n raman
  155  k get nodes
  156  k describe node w1
  157  clear
  158  k get pods -n raman
  159  k logs nginxpod -n raman
  160  kubectl exec -it nginxpod -- /bin/bash
  161  kubectl exec -it nginxpod -- /bin/bash -n raman
  162  kubectl exec -it nginxpod -n raman -- /bin/bash
  163  k get pods -A
  164  k exec -it apache -n karun -- /bin/bash
  165  k exec -it apache -n karun -- ls
  166  k exec -it apache -n karun -- mkdir raman
  167  k exec -it apache -n karun -- ls


====================================================

multiconpod :
    
    
    root@master:~/raman# cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multiconpod
  namespace: raman
spec:
  containers:
  - name: con1
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  - name: con2
    image: redis
    ports:
    - containerPort: 6379




 k delete pods --all -n raman
  181  clear
  182  cat pod
  183  cat pod.yaml
  184  mkdir raman
  185  cd raman/
  186  cp ../pod.yaml .
  187  ls
  188  vi pod.yaml
  189  k create -f pod.yaml
  190  k get pods -nr aman
  191  k get pods -n raman
  192  k get pods -n raman -o wide
  193  k describe pod multiconpod
  194  k describe pod multiconpod -n raman
  195  clear
  196  k get pods -n raman
  197  k exec -it multiconpod -c con1 -- /bin/bash
  198  k exec -it multiconpod -c con1 -n raman -- /bin/bash
  199  k exec -it multiconpod -c con2 -n raman -- /bin/bash
  200  k exec -it multiconpod -c con2 -n raman -- ls
  201  k exec -it multiconpod -c con1 -n raman -- ls


=====================================================================

deployment :

k delete pods --all -n raman
  211  k create deploy dep1 --image httpd --replicas 5 -n raman
  212  k get pods
  213  k get pods -n raman
  214  k get rs -n raman
  215  clear
  216  k get deploy
  217  k get deploy -n raman
  218  k get rs
  219  k get rs -n raman
  220  k get pods -n raman
  221  k delete pod dep1-7fd854dc89-np7xr
  222  k delete pod dep1-7fd854dc89-np7xr -n raman
  223  k get pods -n raman
  224  clear
  225  k get pods
  226  k get pods -n raman
  227  k scale deploy dep1 -n raman --replicas 1
  228  k get pods
  229  k get pods -n raman
  230  k scale deploy dep1 -n raman --replicas 3
  231  k get pods -n raman
  232  k delete pod -n raman dep1-7fd854dc89-kmcb4
  233  k get pods -n raman
  234  k get pods -n raman -o wide
  235  k delete po -n raman dep1-7fd854dc89-k4krq
  236  k get pods -n raman -o wide

========================================================


rollout/ rollbck :
    
    
    Kubernetes Deployment Startegy -- Rolling Update and Recreate:

vi deploy.yaml                                         #nginx:1.14.1
 1248  kubectl apply -f deploy.yaml --record=true
 1249  kubectl rollout status deployment nginx-deployment
 1250  kubectl rollout history deployment nginx-deployment
 1251  vi deploy.yaml                                  #nginx:1.14.2
 1252  kubectl apply -f deploy.yaml --record=true
 1253  kubectl rollout status deployment nginxdep
 1254  kubectl rollout history deployment nginxdep
 1255  vi deploy.yaml                                  #nginx:latest
 1256  kubectl apply -f deploy.yaml --record=true
 1257  kubectl rollout status deployment nginxdep
 1258  kubectl rollout history deployment nginxdep
kubectl describe deployment
kubectl rollout undo deployment nginxdep --to-revision=1
 1292  kubectl rollout status deploy nginxdep


root@tech-master:/gagan# cat deploy.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gagan-app
  labels:
    app: web2-prod-app
spec:
  replicas: 3
  strategy:
    type: Recreate
#    type: RollingUpdate
#    rollingUpdate:
#      maxSurge: 25%
#      maxUnavailable: 25%
  selector:
    matchLabels:
      app: web2-prod-app
  template:
    metadata:
      labels:
        app: web2-prod-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80





1.14.1 :::  
  rs (   nginx-deployment-58887d9567 )  
  
  nginx-deployment-58887d9567-5kqd8   1/1     Running   0          113s
nginx-deployment-58887d9567-gfwt6   1/1     Running   0          113s
nginx-deployment-58887d9567-n8fff 








 nginx:1.14.2 :


rs : NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-5756f889b8   3         3         3       70s
nginx-deployment-58887d9567   0         0         0       8m58s


nginx-deployment-5756f889b8-kzrds   1/1     Running   0          40s
nginx-deployment-5756f889b8-mvbph   1/1     Running   0          37s
nginx-deployment-5756f889b8-vz4wx   1/1 













nginx:latest



nginx-deployment-5756f889b8   0         0         0       3m28s
nginx-deployment-58887d9567   0         0         0       11m
nginx-deployment-5bc54bf6c9   3         3         3       9s





===============================================================

root@tech-master:/gagan# cat pod1.yml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod1
spec:
  containers:
    - name: container1
      image: httpd:2.4
    - name: container2
      image: redis
---
apiVersion: v1
kind: Pod
metadata:
  name: app-pod2
spec:
  containers:
    - name: container1
      image: httpd:2.4
      resources:
        requests:
          cpu: ".2"
          memory: "100M"
        limits:
          cpu: "2"
          memory: "200M"
root@tech-master:/gagan#



============================================


 k get pods -n raman -o wide
  918  curl 192.168.190.68
  919  clear
  920  k delete -f deploy.yml
  921  clear
  922  ls
  923  k create deploy dep1 --image httpd --replicas 3
  924  k get pods
  925  k delete deploy --all
  926  k create deploy dep1 --image httpd --replicas 3
  927  clear
  928  k get pods
  929  k get pods -o wide
  930  k get svc
  931  k expose deploy dep1 --name extsvc --type NodePort  --port 80 --target-port 80
  932*
  933  curl 10.99.79.2
  934  clear
  935  k get pods -o wide
  936  k scale deploy dep1 --replicas 2
  937  k get pods -o wide
  938  k get svc
  939  k scale deploy dep1 --replicas 1
  940  k get pods -o wide
  941  clear
  942  k get pods -o wide
  943  k scale deploy dep1 --replicas 3
  944  k get pods
  945  k get pods -o wide
  946  k expose deploy dep1 --name extsvc --type LoadBalancer  --port 80 --target-port 80
  947  k expose deploy dep1 --name extsvc2 --type LoadBalancer  --port 80 --target-port 80
  948  k get svc


=======================================================================

application code (secrets)    >> dockerfile >>    custom docker image       >>>   deployment  (pods)



SECRETS

    encrypt the data first of all
password :
   echo 'ramankhanna123' | base64       

username :
   echo 'ramankhanna' | base64

  462  cat secret.yaml
  463  kubectl apply -f secret.yaml
  464  kubectl get secret
  465  kubectl describe secret my-secrets
  466  kubectl describe secret my-secrets -o yaml
  467  vi secret-pod.yaml
  468  kubectl apply -f secret
  469  kubectl apply -f secret-pod.yaml
  470  kubectl get pods
  471  kubectl exec -it myapp-pod1 -- /bin/bash
  472  kubectl exec -it myapp-pod2 -- /bin/bash
     #to access username and password as env variables in pod2 :
       echo $SECRET_USERNAME
       echo $SECRET_PASSWD

  473  cat secret-pod.yaml
  474  cat secret.yaml
  475  history
  
  
  
root@tech-master:/gagan# cat secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secrets
type: Opaque
data:
  username: cmFtYW5raGFubmEK
  password: cmFtYW5raGFubmExMjMK
  
  
  
  
  
  
  root@master:~/raman# cat pod2.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod1
  labels:
    app: myapp
spec:
  containers:
  - name: httpd-container
    image: httpd
    volumeMounts:
    - name: cred
      mountPath: /tmp/creds
      readOnly: true
  volumes:
  - name: cred
    secret:
      secretName: my-secrets

  


history :
    
     vi secrets.yml
  961  k getsecrets
  962  k get secrets
  963  cat secrets.yml
  964  k create -f secrets.yml
  965  k get secrets
  966  ls
  967  rm -rf secrets.yml
  968  ls
  969  k describe secret my-secrets
  970  ls
  971  vi pod2.yml
  972  k describe secret
  973  ls
  974  k create -f pod2.yml
  975  k get pods
  976  ls
  977  k exec -it myapp-pod1 -- /bin/bash

    
    
    
    ---------------- as env variables ::
        
      root@master:~/raman# cat pod3.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod2
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: httpd-container
    image: httpd
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secrets
          key: username
    - name: SECRET_PASSWD
      valueFrom:
        secretKeyRef:
          name: my-secrets
          key: password



 cat pod3.yml
  985  k describe secrets
  986  ls
  987  k create -f pod3.yml
  988  k get pods
  989  k exec -it myapp-pod2 -- /bin/bash
  

===========================================



CONFIGMAPS:

echo "Hello from production" > prod.html
echo "Hello from dev" > dev.html
kubectl get configmaps
kubectl create configmap prod.cmap --from-file=prod.html
kubectl create configmap dev.cmap --from-file=dev.html
kubectl get configmaps
kubectl get configmap dev.cmap -o yaml
kubectl get configmap prod.cmap -o yaml
vi pods.yaml

----

root@gagan-master:/gagan/cmap# cat pods.yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-nginx
  labels:
    app: prod-nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: /usr/share/nginx/html
  volumes:
    - name: config-volume
      configMap:
        name: prod.cmap
        items:
        - key: prod.html
          path: index.html





history fr reference :
    
    
     echo "Hello from production" > prod.html
  996  ls
  997  echo "Hello from dev" > dev.html
  998  kubectl get configmaps
  999  k create -h
 1000  k create cm prod.cmap -h
 1001  clear
 1002  k create cm prod.cmap --from-file=prod.html
 1003  kubectl create configmap dev.cmap --from-file=dev.html
 1004  k get cm
 1005  k describe cm prod.cmap
 1006  clear
 1007  k get cm prod.cmap -o yaml
 1008  clear
 1009  ls
 1010  vi pod4.yml
 1011  k create -f pod4.yml
 1012  k get pods
 1013  k get cm
 1014  k describe pod prod-nginx
 1015  clear
 1016  k get pods
 1017  k get pods -o wide
 1018  curl 192.168.80.195
 1019  vi pod4.yml
 1020  k get cm
 1021  k describe cm dev.cmap
 1022  k apply -f pod4.yml
 1023  k get pods
 1024  k get pods -o wide
 1025  curl 192.168.80.195
 1026  k replace -f pod4.yml
 1027  k get pods
 1028  vi pod4.yml
 1029  k delete pod prod-nginx
 1030  k create -f pod4.yml
 1031  k get pods -o wide
 1032  curl 192.168.80.201
 1033  cat pod4.yml
 1034  k get cm
 1035  k edit cm dev.cmap
 1036  k get pods -o wide
 1037  curl 192.168.80.201
 1038  k get cm
 1039  k edit cm dev.cmap
 1040  k get pods -o wide
 1041  curl 192.168.80.201
 1042  clear
 1043  k get cm
 1044  k describe cm dev.cmap
 1045  cat pod4.yml
 1047  k expose pod prod-nginx --name cmextsvc --type NodePort --target-port 80 --port 80
 1048  k get svc
 1049  clear
 1050  k get cm
 1051  k edit cm dev.cmap
 1052  k get pods -o wide
 1053  curl 192.168.80.201
 1054  clear
 1055  k describe cm
 1056  curl 192.168.80.201


=======================================================================================

ingress conroller setup :

Note : make sure latest aws cli installed ..

alias k=kubectl
   22  k get nodes

helm insall :

   23  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   24  $ chmod 700 get_helm.sh
   25  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
   26  chmod 700 get_helm.sh
   27  ./get_helm.sh
   28  k ge ns
   29  k get ns
   30  k create ns ingress
   31  clear

   32  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   33  helm repo update
 
   35  helm list
   36  helm repo list


Note : below command will work when made change in kubeconfig file : apiVersion: client.authentication.k8s.io/v1beta1

helm install my-release ingress-nginx/ingress-nginx \
 --namespace ingress \
 --set controller.replicaCount=2 \
 --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
 --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

   
   40  ls -a
   41  cd .kube/
   42  ls
   43  cat config
   44  vi config
   45  k get pods
   46  k get nodes
   47  vi config


---- execute below deploymnt , service , ingress resource : in ingress ns

root@master:~/raman# cat ingdeploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-app
  labels:
    app: test-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-app
  template:
    metadata:
      labels:
        app: test-app
    spec:
      containers:
      - name: test-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 50m
            memory: 64Mi
---
apiVersion: v1
kind: Service
metadata:
  name: raman-service
spec:
  type: NodePort
  ports:
  - port: 80
  selector:
    app: test-app
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-app-ingress
  namespace: ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /raman(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: raman-service
            port:
              number: 80




--- k create -f ingdeploy.yaml -n ingress


---- now try to add more deploymnt , add a service and add a new path in ingress to get a new api on alb dns

=============================================

pv /pvc 



root@master:~/raman# cat pod5.yml
apiVersion: v1
kind: Pod
metadata:
  name: redispod
  namespace: default
spec:
  restartPolicy: Never
  volumes:
  - name: vol
    hostPath:
      path: /home/ubuntu
  containers:
  - name: redis
    image: "redis"
    volumeMounts:
    - name: vol
      mountPath: /hostfile
---
apiVersion: v1
kind: Pod
metadata:
  name: redispod1
  namespace: default
spec:
  restartPolicy: Never
  volumes:
  - name: vol
    hostPath:
      path: /home/ubuntu
  containers:
  - name: redis
    image: "redis"
    volumeMounts:
    - name: vol
      mountPath: /hostfile
  nodeName: w1


--------------------------------------------------------------------------------------





root@master:~/raman# cat pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - w2  # Replace with the name of the node where /mnt/data exists
                
                
                history for reference :
                    
                    k delete pods --all
 2124  clear
 2125  ls
 2126  vi pv.yaml
 2127  k create -f pv.yaml
 2128  k get pv
 2129  vi pvc.yml
 2130  k delete pv --all
 2131  vi pv.yaml
 2132  cat pv
 2133  cat pv.yaml
 2134  wq!
 2135  k create -f pv.yaml
 2136  k get pv
 2137  clear
 2138  ls
 2139  k get pv





                
                
                
root@master:~/raman# cat pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
  namespace: raman
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage




vi pvc.yml
 2142  k create -f pvc.yml
 2143  k get pvc
 2144  k get pvc -n raman
 2145  k get pv

-------------------------------------------------------------


root@master:~/raman# cat persientdeploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: raman
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app-container
        image: nginx
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: rk
      volumes:
      - name: rk
        persistentVolumeClaim:
          claimName: local-pvc


============================================


external storage as pv :

Step-by-Step Guide for Setting Up NFS-based Persistent Storage in Kubernetes
This guide walks through setting up an NFS-based PersistentVolume (PV) and PersistentVolumeClaim (PVC) in Kubernetes, deploying an Nginx-based application using this storage, and exposing the deployment via a NodePort service.
1. Install Required NFS Packages on Nodes
Ensure that the worker nodes in the Kubernetes cluster can mount the NFS storage.
For Ubuntu/Debian:
bash
CopyEdit
sudo apt-get update
sudo apt-get install -y nfs-common
For RHEL/CentOS:
bash
CopyEdit
sudo yum install -y nfs-utils
2. Mount NFS Manually on the Node
Verify that the node can connect to the NFS server.
Create a mount directory:
bash
CopyEdit
sudo mkdir -p /efs
Mount the NFS share:
bash
CopyEdit
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 172.31.12.142:/ /efs
Check if the mount is successful:
bash
CopyEdit
df -h
List the contents of the mounted directory:
bash
CopyEdit
cd /efsls -l
3. Define Persistent Volume (PV) and Persistent Volume Claim (PVC)
Create a YAML file (nfs-pv-pvc.yaml) with the following contents:
yaml
CopyEdit
apiVersion: v1kind: PersistentVolumemetadata:
  name: nfs-websitespec:
  capacity:
    storage: 11Mi
  accessModes:
    - ReadWriteMany
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /
    server: 172.31.12.142
---apiVersion: v1kind: PersistentVolumeClaimmetadata:
  name: nfs-demospec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Mi
  volumeName: nfs-website
Apply the PV and PVC:
bash
CopyEdit
kubectl apply -f nfs-pv-pvc.yaml
Verify that the PV and PVC are created:
bash
CopyEdit
kubectl get pv
kubectl get pvc
4. Deploy an Nginx Application Using the NFS Storage
Create a YAML file (nfs-nginx-deployment.yaml) with the following contents:
yaml
CopyEdit
apiVersion: apps/v1kind: Deploymentmetadata:
  name: nfs-ramanspec:
  replicas: 10
  selector:
    matchLabels:
      role: nfs-raman
  template:
    metadata:
      labels:
        role: nfs-raman
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nfs
          mountPath: /usr/share/nginx/deploydata
      volumes:
      - name: nfs
        persistentVolumeClaim:
          claimName: nfs-demo
Deploy Nginx:
bash
CopyEdit
kubectl apply -f nfs-nginx-deployment.yaml
Verify Pods:
bash
CopyEdit
kubectl get pods
5. Expose the Nginx Deployment as a NodePort Service
bash
CopyEdit
kubectl expose deploy nfs-raman --name nfs-svc --type NodePort --target-port 80 --port 80
Check the service details:
bash
CopyEdit
kubectl get svc nfs-svc
Look for the NodePort assigned (e.g., :3XXXX).
6. Configure Security Groups (AWS EC2/NFS)
Ensure that the security group of the NFS server allows incoming traffic on the required ports:
Allow NFS traffic: Port 2049 (TCP) from worker nodes.
Allow HTTP traffic: Port 80 from external clients.
Allow NodePort range: (e.g., 30000-32767) if accessing via NodePort.7. Access the Application
Find the public IP of a worker node:
bash
CopyEdit
curl ifconfig.me
or check in your cloud provider’s dashboard.
Browse the Application
Open a browser and enter:
php-template
CopyEdit
http://<worker-node-public-ip>:<NodePort>
For example:
cpp
CopyEdit
http://18.222.45.67:31080
8. Verify NFS Storage is Working
Inside one of the running Nginx pods:
bash
CopyEdit
kubectl exec -it <nginx-pod-name> -- /bin/sh
Check the mounted directory:
sh
CopyEdit
cd /usr/share/nginx/deploydatals -l
Any changes in /efs on the NFS server should reflect inside the pod.


==================================================================


RBAC :


Lab Activity: Enforce Namespace-Specific Permissions and Test Access
Step 1: Create a Namespace
Create a new namespace for testing:

bash
Copy
kubectl create namespace test-namespace
Verify the namespace:

bash
Copy
kubectl get namespaces
Step 2: Create a Role
Create a Role that allows read-only access to Pods in the test-namespace:

Write the following YAML to a file (pod-reader-role.yaml):

yaml
Copy
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: test-namespace
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
Apply the Role:

bash
Copy
kubectl apply -f pod-reader-role.yaml
Verify the Role:

bash
Copy
kubectl get roles -n test-namespace
Step 3: Create a ServiceAccount
Create a ServiceAccount in the test-namespace:

bash
Copy
kubectl create serviceaccount test-sa -n test-namespace
Verify the ServiceAccount:

bash
Copy
kubectl get serviceaccounts -n test-namespace
Step 4: Create a RoleBinding
Bind the pod-reader Role to the test-sa ServiceAccount:

Write the following YAML to a file (pod-reader-rolebinding.yaml):

yaml
Copy
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: test-namespace
subjects:
- kind: ServiceAccount
  name: test-sa
  namespace: test-namespace
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
Apply the RoleBinding:

bash
Copy
kubectl apply -f pod-reader-rolebinding.yaml
Verify the RoleBinding:

bash
Copy
kubectl get rolebindings -n test-namespace
Step 5: Test Access Using kubectl auth can-i
Authenticate as the ServiceAccount:

Use the --as flag to impersonate the test-sa ServiceAccount and test access.

Test Permissions:

Check if the ServiceAccount can list Pods in the test-namespace:

bash
Copy
kubectl auth can-i list pods --namespace test-namespace --as system:serviceaccount:test-namespace:test-sa
Expected Output: yes.

Check if the ServiceAccount can create Pods in the test-namespace:

bash
Copy
kubectl auth can-i create pods --namespace test-namespace --as system:serviceaccount:test-namespace:test-sa
Expected Output: no (since the Role only allows get, list, and watch).

Check if the ServiceAccount can list Pods in the default namespace:

bash
Copy
kubectl auth can-i list pods --namespace default --as system:serviceaccount:test-namespace:test-sa
Expected Output: no (since the Role is namespace-specific).



=====================================================================================================


blue-green deploymnt :
    
    blue : old version : nginx 
    green : newer version : httpd
    
    
    
    root@master:~/raman# cat deployblue.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deployment
  namespace: raman
  labels:
    deploy: dep1
spec:
  replicas: 3
  selector:
    matchLabels:
      rk: test
  template:
    metadata:
      labels:
        rk: test
    spec:
      containers:
      - name: nginxcon
        image: nginx:latest
        ports:
        - containerPort: 80

root@master:~/raman# cat deploygreen.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deployment
  namespace: raman
  labels:
    deploy: dep1
spec:
  replicas: 3
  selector:
    matchLabels:
      rk: test2
  template:
    metadata:
      labels:
        rk: test2
    spec:
      containers:
      - name: httpdcon
        image: httpd
        ports:
        - containerPort: 80

root@master:~/raman# cat service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: raman
spec:
  type: NodePort
  selector:
     rk: test2
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007




# a/b deploymnt
create a deployment blue-nginx and expose it as a nodeport with labels for pods as app=nginx

service created has labels as app=nginx

this setup is reflected all blue-nginx pods to th users

,, now create other deploymnt green-httpd with labels for pods as app=httpd
instead of exposing this with new service , change the labels of service to app =httpd 

now this setup will reflect all green-httpd pods to users



--------------------------------------------------------------------------------------------------------------------------------------


canary :
    
    
    # canary deploymnt :
 

create deploymnt app1 of nginx 

create deploy app2 of httpd

change labels in both deploy to be app=canary

create a service having labels app=canary to capture every pod of each deploymnt

now adjust and scale of % pods accordingly 


 vi deploygreen.yml
 2305  k apply -f deploygreen.yml
 2306  k get deploy -n raman
 2307  clear
 2308  k get pods -n raman
 2309  k scale deploy blue-deployment -n raman --replicas 7
 2310  k scale deploy green-deployment -n raman --replicas 3
 2311  k get pods -n raman
 2312  k scale deploy green-deployment -n raman --replicas 5
 2313  k scale deploy blue-deployment -n raman --replicas 5
 2314  clear


======================================================================

hpa 

note : metric serve shud b thre 
resource limitation shud b thre...

 k get hpa -A
 2004  k create deploy dep1 -n raman --image httpd --replicas 1
 2005  k get pods -n raman
 2006  k top pods
 2007  k top nodes
 2009  k get pods -n kube-system


-- install a metric server functionality :
   
    cd raman/
 2013  ls
 2014  git clone https://github.com/ramannkhanna2/k8s_metrics_server.git
 2015  ls
 2016  cd k8s_metrics_server/
 2017  ls
 2018  cd deploy/
 2019  ls
 2020  cd 1.8+/
 2021  ls
 2022  k apply -f .
 2023  k get pods -n kube-system

    
    ------------------------------------------------------------
    
    
    root@master:~/raman# cat deploy.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu-deployment
  namespace: raman
  labels:
    deploy: dep1
spec:
  replicas: 1
  selector:
    matchLabels:
      rk: test
  template:
    metadata:
      labels:
        rk: test
    spec:
      containers:
      - name: ubuntucon
        image: ubuntu
        command: ["sleep","infinity"]
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "150m"
        ports:
        - containerPort: 80


------------------------------------------------

 k autoscale deploy ubuntu-deployment --cpu-percent=50 --min=1 --max=5 -n raman
 2060  k get hpa
 2061  k get hpa -n raman

------------------------------------------


 2063  k get hpa -n raman -w
 

run a new session and see if inc load on a pod increases them automat !!

-=======================================================






In Kubernetes, Node Selectors, Affinity & Anti-affinity, and Taints & Tolerations are mechanisms to control pod scheduling on nodes.
1. Node Selectors, Affinity, and Anti-affinity
These mechanisms help in placing pods on specific nodes based on labels.
1.1 Node Selector
The simplest way to schedule pods on specific nodes.
Requires labels on nodes and a matching nodeSelector in the pod specification.
Example:
yaml
CopyEdit
apiVersion: v1kind: Podmetadata:
  name: my-podspec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: nginx
    image: nginx
If a node has the label disktype=ssd, the pod will be scheduled on that node.
Limitations of NodeSelector
Can only specify exact matches (key=value).
No advanced expressions (like OR, NOT, or weight-based preference).1.2 Node Affinity
A more flexible way than nodeSelector using rules with operators.
Defined under nodeAffinity in spec.affinity.
Has two types:
Required (hard constraint) → requiredDuringSchedulingIgnoredDuringExecution (Pod must be placed on a node that meets the condition)
Preferred (soft constraint) → preferredDuringSchedulingIgnoredDuringExecution (Pod will try to be placed on a node, but if no matching node is found, it can still be scheduled)
Example (Hard Constraint - Required)
yaml
CopyEdit
apiVersion: v1kind: Podmetadata:
  name: my-podspec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
            - nvme
  containers:
  - name: nginx
    image: nginx
The pod will only be scheduled on nodes labeled disktype=ssd or disktype=nvme.Example (Soft Constraint - Preferred)
yaml
CopyEdit
apiVersion: v1kind: Podmetadata:
  name: my-podspec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 10
          preference:
            matchExpressions:
            - key: region
              operator: In
              values:
              - us-east
  containers:
  - name: nginx
    image: nginx
Kubernetes will prefer scheduling this pod on nodes with region=us-east, but it won't fail if no such node exists.1.3 Pod Affinity & Anti-affinity
These define where pods should be scheduled based on other running pods.
Pod Affinity → Prefer placing pods on the same node as another set of pods.
Pod Anti-affinity → Prefer placing pods on different nodes from another set of pods.Example (Pod Affinity)
yaml
CopyEdit
apiVersion: v1kind: Podmetadata:
  name: web-podspec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - backend
          topologyKey: "kubernetes.io/hostname"
  containers:
  - name: web
    image: nginx
This pod will be scheduled on the same node as another pod with the label app=backend.Example (Pod Anti-affinity)
yaml
CopyEdit
apiVersion: v1kind: Podmetadata:
  name: web-podspec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - web
          topologyKey: "kubernetes.io/hostname"
  containers:
  - name: web
    image: nginx
Ensures that no two pods with label app=web are scheduled on the same node.2. Taints and Tolerations
Taints and tolerations prevent pods from being scheduled on nodes unless they explicitly tolerate the taint.
2.1 Taints (Node Level)
Taints are applied to nodes to repel pods that do not tolerate them.
Example:
sh
CopyEdit
kubectl taint nodes node1 key=value:NoSchedule
This ensures that only pods with the corresponding toleration can be scheduled on node1.
2.2 Tolerations (Pod Level)
Tolerations allow pods to be scheduled on nodes with matching taints.
Example:
yaml
CopyEdit
apiVersion: v1kind: Podmetadata:
  name: toleration-podspec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
This pod can be scheduled on a node with key=value:NoSchedule.
Taint EffectsEffectDescriptionNoSchedulePods that don't tolerate the taint will not be scheduled on the node.PreferNoScheduleKubernetes will try to avoid scheduling pods on the node, but may still do so.NoExecuteExisting pods will be evicted if they don't tolerate the taint.Key DifferencesFeaturePurposeApplied ToConstraintsNode SelectorSchedule pods on specific nodesPodExact match onlyNode AffinityMore flexible node selectionPodSupports expressions & preferencesPod AffinitySchedule pods near each otherPodRequires labels & topologyKeyPod Anti-affinitySpread pods across nodesPodRequires labels & topologyKeyTaints & TolerationsRestrict schedulingNode (Taint), Pod (Toleration)Used for node isolation


-----------------------------------------======================================================================



Step 1: Install etcd Client
To interact with etcd, you need to install the etcd client:
bash
CopyEdit
apt install etcd-client
Then, export the API version:
bash
CopyEdit
export ETCDCTL_API=3
This ensures that the commands run with etcd v3, which is the latest version.
Step 2: Find the Static Pod File Path
To properly restore etcd, you need to find its configuration files.
Check Kubelet Process:
bash
CopyEdit
ps -ef | grep kubelet
This helps you find where the kubelet process is running.
Navigate to Kubelet Directory:


bash
CopyEdit
cd /var/lib/kubelet/ls
This lists the contents of the directory where static pod configurations are stored.
Check Kubelet Config File:
bash
CopyEdit
cat config.yaml
This file contains the configuration settings for kubelet, which includes the location of important system files.
Step 3: Take an ETCD Backup
Run the backup command:
bash
CopyEdit
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /root/myclust.db
--endpoints=https://127.0.0.1:2379: Connects to the local etcd instance.
--cacert: Specifies the certificate authority file for authentication.
--cert: Specifies the client certificate for authentication.
--key: Specifies the private key for authentication.
snapshot save /root/myclust.db: Saves the snapshot to the specified location.
Verify the backup
bash
CopyEdit
ls -l /root/myclust.db
This ensures that the backup file exists.
Step 4: Delete All Pods
To simulate data loss or failure, delete all pods in the cluster:
bash
CopyEdit
kubectl delete pods --all -n <namespace>
This removes all running pods, demonstrating a failure scenario.
Step 5: Restore the ETCD Backup
Find the Original Data Directory:
bash
CopyEdit
ps -ef | grep etcd
You should see an entry pointing to the etcd process. Look for the --data-dir argument, which tells you where etcd stores its data. Usually, it is:
bash
CopyEdit
/var/lib/etcd
Restore the Backup to a New Directory
bash
CopyEdit
etcdctl --data-dir /var/lib/etcd-new snapshot restore /root/myclust.db
--data-dir /var/lib/etcd-new: Specifies a new directory where the snapshot will be restored.
The command automatically creates /var/lib/etcd-new/etcd inside the specified directory. You do not need to create this manually.
Step 6: Update the ETCD Manifest File
Since etcd runs as a static pod, you need to update the etcd.yaml manifest file to point to the new data directory.
Edit the manifest file:
bash
CopyEdit
vi /etc/kubernetes/manifests/etcd.yaml
Find the volumes section and update the path:
yaml
CopyEdit
volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-new
      type: DirectoryOrCreate
    name: etcd-data
Change path: /var/lib/etcd to path: /var/lib/etcd-new.
Save and exit (Esc, then :wq in vi editor).Step 7: Wait for ETCD to Restart and Verify
After making the changes, the etcd pod will automatically restart since the kubelet continuously watches the manifest files.
Wait for some time and check if etcd is running:
bash
CopyEdit
kubectl get pods -n kube-system
The etcd pod should restart and run successfully.
Verify that the backup restore was successful
bash
CopyEdit
kubectl get pods --all-namespaces
If your original pods start appearing again, the restoration was successful.
Summary
Installed the etcd-client.
Found the etcd data directory.
Took a snapshot backup of etcd.
Deleted all pods to simulate data loss.
Restored etcd from the backup to a new directory.
Updated the etcd static pod manifest to use the new data directory.
Waited for Kubernetes to restart etcd and verified that all pods were back.

========================================================================


Troubleshooting Cluster Component Failures
Kubernetes clusters rely on several key components: kubelet, scheduler, and API server. When any of these components fail, the cluster may become unstable or unresponsive. Let’s break down how to analyze failures, monitor logs, and conduct lab activities for troubleshooting.
1. Analyzing kubelet, scheduler, and API server failuresa. Kubelet Failure Analysis
The kubelet is responsible for managing pods on a node and ensuring containers run as expected. If the kubelet fails, the node may become NotReady or fail to schedule workloads.
Steps to Analyze Kubelet Failures:
Check Node Status:
bash
CopyEdit
kubectl get nodes
If the node is NotReady, kubelet might be down.
Inspect kubelet Service:
bash
CopyEdit
systemctl status kubelet
If it's inactive (dead), try restarting it:
bash
CopyEdit
systemctl restart kubelet
Check kubelet Logs for Errors:
bash
CopyEdit
journalctl -u kubelet -f --no-pager
Common kubelet issues:
Certificate issues: /var/lib/kubelet/pki
Misconfigured kubelet.conf
Disk pressure issues
Pod sandbox errors
b. Scheduler Failure Analysis
The kube-scheduler is responsible for assigning pods to nodes. If it fails, new workloads won’t be scheduled.
Steps to Troubleshoot:
Check Scheduler Status:
bash
CopyEdit
kubectl get pods -n kube-system | grep scheduler
The pod should be Running. If it’s in CrashLoopBackOff or Pending, there’s an issue.
Check Logs for Errors:
bash
CopyEdit
kubectl logs -n kube-system kube-scheduler-<pod-name>
Look for errors related to:
Scheduler policies
Authentication issues
APIServer connectivity issues
Restart the Scheduler:
bash
CopyEdit
kubectl delete pod kube-scheduler-<pod-name> -n kube-system
This forces the scheduler to restart.
c. API Server Failure Analysis
The API server is the core of the Kubernetes control plane. If it fails, the cluster becomes unresponsive.
Steps to Troubleshoot API Server:
Check API Server Pod Status:
bash
CopyEdit
kubectl get pods -n kube-system | grep apiserver
If the API server pod is missing or CrashLoopBackOff, it indicates a failure.
Check API Server Logs:
bash
CopyEdit
kubectl logs -n kube-system kube-apiserver-<pod-name>
Common issues:
Certificate expiration (/etc/kubernetes/pki)
ETCD connection errors
Misconfigured flags in /etc/kubernetes/manifests/kube-apiserver.yaml
Restart API Server (If necessary):
bash
CopyEdit
systemctl restart kube-apiserver
2. Monitoring Cluster Logs for Failures
Kubernetes generates logs for each component, which helps in debugging failures.
a. Cluster-wide Event Logs
bash
CopyEdit
kubectl get events --sort-by=.metadata.creationTimestamp
Shows all failures and warnings across the cluster.b. Node Logs
bash
CopyEdit
kubectl describe node <node-name>
Provides detailed information about node health and kubelet status.c. Pod Logs
bash
CopyEdit
kubectl logs <pod-name> -n <namespace>
Shows logs of individual pods.d. ETCD Logs (for API Server Issues)
bash
CopyEdit
journalctl -u etcd -f --no-pager
ETCD errors may cause API server failures.3. Lab ActivitiesActivity 1: Simulate and Recover from API Server Failure
Goal: Intentionally stop the API server and bring it back online.
Steps:
Simulate API Server Failure:
bash
CopyEdit
systemctl stop kube-apiserver
Now, try running:
bash
CopyEdit
kubectl get nodes
It will fail because the API server is down.
Recover API Server:
bash
CopyEdit
systemctl start kube-apiserver
Verify API Server Health:
bash
CopyEdit
kubectl get componentstatuses
Activity 2: Debug a Failing Node using Kubelet Logs
Goal: Investigate a failing node where pods are not running properly.
Steps:
Identify the failing node:
bash
CopyEdit
kubectl get nodes
Check kubelet logs for the node:
bash
CopyEdit
journalctl -u kubelet -f --no-pager
Look for common errors:
failed to start container: Docker/CRI issue.
TLS handshake error: Certificate issue.
disk pressure condition: Low disk space.
Fix the issue (Example - Disk Pressure):
bash
CopyEdit
df -h  # Check disk spacerm -rf /var/log/*  # Clear old logs
systemctl restart kubelet

===================================================================================



Troubleshooting Application Failures in Kubernetes1. Debugging Failing Pods
When an application fails in Kubernetes, debugging involves analyzing Events, Probes, and Logs.
a. Check Pod Events
Events provide insights into scheduling failures, image pull errors, and other issues.
sh
CopyEdit
kubectl describe pod <pod-name>
Look for errors under Events, such as:
Failed scheduling
Image pull errors
Liveness/Readiness probe failures
b. Inspect Container Logs
Pod logs help identify runtime errors.
sh
CopyEdit
kubectl logs <pod-name> -n <namespace>
If the pod has multiple containers:
sh
CopyEdit
kubectl logs <pod-name> -c <container-name>
For previous logs (e.g., if the pod restarted):
sh
CopyEdit
kubectl logs --previous <pod-name>
c. Debug with kubectl exec
Run commands inside a running container.
sh
CopyEdit
kubectl exec -it <pod-name> -- /bin/sh
Check environment variables:
sh
CopyEdit
env
Inspect filesystem:
sh
CopyEdit
ls -l /appcat /etc/config.yaml
2. Common Application Issues & Fixes
Here are common problems and how to address them:
a. Misconfigurations
Wrong environment variables: Ensure correct config maps and secrets are mounted.
Incorrect ports: Verify container and service ports match.
Wrong file paths: Check mounted volumes.b. CrashLoops (CrashLoopBackOff)
Occurs when a container starts but crashes repeatedly.
sh
CopyEdit
kubectl get pod <pod-name> -o wide
kubectl describe pod <pod-name>
Fixes:
Check logs for app errors.
Verify health probes (livenessProbe, readinessProbe).
Ensure dependencies (databases, APIs) are accessible.
Increase startup delay if the app takes time to initialize.c. Image Pull Errors
sh
CopyEdit
kubectl describe pod <pod-name> | grep "Failed"
Fixes:
Ensure the image exists in the registry.
Authenticate to private registries correctly.
Check for typos in the image name and tag.3. Lab Activity: Debugging a Misconfigured ApplicationStep 1: Deploy a Broken App
Create a faulty deployment (faulty-app.yaml):
yaml
CopyEdit
apiVersion: apps/v1kind: Deploymentmetadata:
  name: faulty-appspec:
  replicas: 1
  selector:
    matchLabels:
      app: faulty-app
  template:
    metadata:
      labels:
        app: faulty-app
    spec:
      containers:
        - name: faulty-app
          image: nginx:latest
          ports:
            - containerPort: 8080  # Wrong port (NGINX runs on 80)
          livenessProbe:
            httpGet:
              path: /
              port: 8080  # Wrong port, probe will fail
            initialDelaySeconds: 5
            periodSeconds: 10
Step 2: Deploy and Debug
sh
CopyEdit
kubectl apply -f faulty-app.yaml
kubectl get pods
kubectl describe pod <faulty-pod>
kubectl logs <faulty-pod>
Step 3: Fix the Issue
Edit faulty-app.yaml:
Change containerPort from 8080 to 80
Update livenessProbe port to 80
Apply changes:
sh
CopyEdit
kubectl apply -f faulty-app.yaml

==================================================


helm :
    
    helm
 2209  clear
 2210  helm list
 2211  helm repo list
 2212  helm list
 2213  helm repo list
 2214  helm repo add bitnami
 2215  helm repo add bitnami https://charts.bitnami.com/bitnami
 2216  helm repo list
 2217  helm search repo bitnami
 2218  clear
 2219  helm search repo bitnami
 2220  clear
 2221  helm search repo bitnami/jenkins
 2222  helm repo list
 2223  helm repo remove ingress-nginx
 2224  helm repo list
 2225  helm search repo bitnami/tomcat
 2226  helm show values bitnami/tomcat
 2227  clear
 2228  helm show chart bitnami/tomcat
 2229  clear
 2230  helm list
 2231  kubectl get pods
 2232  kubectl delete deploy --all
 2233  helm install raman-tomcat bitnami/tomcat
 2234  clear
 2235  helm list
 2236  kubectl get all
 
 2237  kubect logs deployment.apps/raman-tomcat
 2238  kubectl logs deployment.apps/raman-tomcat
 2239  kubectl describe deployment.apps/raman-tomcat
 2240  clear
 2241  kubectl get pods
 2242  kubectl describe pods
 2243  clear
 2244  kubectl get all
 2245  helm status raman-tomcat
 2246  clear
 2247  helm uninstall raman-tomcat
 2248  kubectl get all
 2249  helm install raman-tomcat bitnami/tomcat --set persistence.enabled=false --set service.type=NodePort
 2250  clear
 2251  helm list
 2252  kubectl get all
 2253  clear
 2254  kubectl get all
 2255  helm uninstall raman-tomcat


===================================================================
```
