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

   # step 2-5
     Deploy Test Application to verify Load balance is working by assiging the IP.
     $ kubectl -n default apply -f web-app-deployment.yml

   # step 2-6
     Verify MetallB assigned an IP address
     $ kubectl -n default get pods
     $ kubectl -n default get services
     
# Step 3
Deploy loadbalancer to overcome the loadbalancer stratgy.

   # Step 3-1
     Install NGINX Ingress Controller with Helm

      Pull to local system.
      $ helm pull oci://ghcr.io/nginxinc/charts/nginx-ingress --untar --version 0.17.1
      
      move in the ingress folder.
      $ cd nginx-ingress

      create a namespace
      $ kubectl create ns nginx-ingress
   
      Apply crds for the nginx-ingress namespace
      $ kubectl apply -f crds -n nginx-ingress

      deploy the ingress in the namespaces.
      $ helm install nginx-ingress oci://ghcr.io/nginxinc/charts/nginx-ingress --version 0.17.1 -n nginx-ingress

   # Step 3-2
      Verify NGINX Ingress Installation

      $ kubectl -n nginx-ingress get pods
      $ kubectl -n nginx-ingress get services

   # step 3-3
      Create an Ingress for the Test Applications
      to test the ingress we need to change the webapp service type from LoadBalancer to default ClusterIP (web app was deployed in the step 2-5)

      create the ingress resource using the file and test using the ingress IP.
      $ kubectl -n default apply -f web-app-ingress.yml
      $ kubectl -n default get ingress


