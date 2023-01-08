# Single Node Kubernetes
Single Node Kubernetes for Nginx & PHP-FPM with [K3s](https://k3s.io/)
> K3s is a highly available, certified Kubernetes distribution designed for production workloads in unattended, resource-constrained, remote locations or inside IoT appliances

## Installation

1. Install K3s using ```curl -sfL https://get.k3s.io | sh - ```

2. Check K3s installation ```kubectl get node```. Take note the node name
    ~~~
    kubectl get node

    NAME                    STATUS   ROLES                  AGE   VERSION
    greencloud.1665793034   Ready    control-plane,master   30d   v1.25.4+k3s1
    ~~~

3. 
