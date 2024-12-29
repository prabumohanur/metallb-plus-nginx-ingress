# metallb-plus-nginx-ingress
Deploy the ingress with loadbalancer in local

# Step 1
Create a local cluster for reference use the below link for cluster deployment.
https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/

# step 2
Install MetallB

Preparation
If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.
Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.
You can achieve this by editing kube-proxy config in current cluster:

kubectl edit configmap -n kube-system kube-proxy

SEARCH THE STRING strictARP: AND CHANGE FROM false to true.
strictARP: true

   # step 2-1
     deploy metallb:
     
     $ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
     
   # step 2-2
     Verify MetallB Installation:
     $ kubectl -n metallb-system get pods
     $ kubectl api-resources| grep metallb

   # step 2-3
     Create IP Pool (make sure you are using your valied ip segment for accessing the app through your network)
     $ kubectl -n metallb-system apply -f pool-1.yml

   # step 2-4
     Create L2Advertisement
     $ kubectl -n metallb-system apply -f l2advertisement.yml

   # Deploy Test Application to verify Load balance is working by assiging the IP.
     $ kubectl -n default apply -f web-app-deployment.yml

   # Verify MetallB assigned an IP address
     $ kubectl -n default get pods
     $ kubectl -n default get services
