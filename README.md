# Single Node Kubernetes for Nginx & PHP-FPM
Single Node Kubernetes for Nginx & PHP-FPM with [K3s](https://k3s.io/)  ```Tested in Ubuntu 22.04 and AlmaLinux 8.4 RAM 2GB+```
> K3s is a highly available, certified Kubernetes distribution designed for production workloads in unattended, resource-constrained, remote locations or inside IoT appliances

## Installation

1. Clone this repository: ```git clone https://github.com/tobing/single-node-kubernetes.git ``` 
    
    >local_persistentvolume.yaml :   To define volume in the local host (/tmp) [link](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
    
    >*_deployment.yaml           :   To define kubernetes Deployment object [link](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)


    >*_service.yaml              :   To define kubernetes Service [link](https://kubernetes.io/docs/concepts/services-networking/service/)
    
    >*_hpa.yaml                  :   To define kubernetes Horizontal Pod Autoscale [link](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
    
    >nginx_configMap.yaml        :   To define config related to app (nginx.conf) [link](https://kubernetes.io/docs/concepts/configuration/configmap/)
    
2. Install K3s using ```curl -sfL https://get.k3s.io | sh - ```

3. Check K3s installation ```kubectl get node```. Take note the node name and paste it to the local_persistentvolume.yaml section ```nodeAffinity.values```

    ```
    NAME                    STATUS   ROLES                  AGE   VERSION
    greencloud.1665793034   Ready    control-plane,master   30d   v1.25.4+k3s1
    ```
    
4. Traefik service installed and running automatically after K3s installation completed. Because Traefik & Nginx service using port 80, we need to delete Traefik service ```kubectl --namespace kube-system delete svc traefik```

5. Go to inside ```single-node-kubernetes``` directory and run ```kubectl apply -f .```

6. Verify all running properly ```kubectl get pods,deploy,svc,pv,pvc,hpa```

    ![image](https://user-images.githubusercontent.com/16585545/211197147-d25dee00-2025-4598-811d-0f6d9c5d4bf7.png)


7. If something wrong check with ```kubectl describe <NAME>```, example
    - For pod       :   ```kubectl describe pod/php-776fc877d8-2q2fq```
    - For service   :   ```kubectl describe service/nginx```  
    
8. Take note the EXTERNAL-IP for ```service/nginx```. The TYPE is [LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

9. Open a web browser and go to http://EXTERNAL-IP. The page should show "403 Forbidden"

10. Because the persistent volume location in /tmp we need to create a test file ```vi /tmp/test.php``` and type code below then save
    ```
    <?php
        echo gethostname();
    ?>
    ```
    
11. Back to the web browser and open http://EXTERNAL-IP/test.php. Reload the page several times and the hostname will changed. This mean requests served by different pod

12. To test Horizontal Pod AutoScale (hpa) need to open 2 shell session
    - First shell to monitor the pods run command ```watch -n1 kubectl get pods```
    - Second shell from same or different server to run ab apache benchmark (change -n value accordingly)
        ```ab -n 100000 -c 1000 http://EXTERNAL-IP/test.php/```
        
    After the ab apache benchmark running for some time, you will see the pods replica increasing in the first shell

13. You can modify 

    - *hpa.yaml in the sections ```minReplicas```, ```maxReplicas```, ```averageUtilization```
    
    - *deployment.yaml in the sections ```resources```
    
    After modified run ```kubectl apply -f .``` then test again and verify

## kubectl command options

- Show pods, deployments, services, persistent volumes, persistent volume claims, horizontal pod autoscale  
    ```kubectl get pods,deploy,svc,pv,pvc,hpa```
- Show pods, deployments, services, persistent volumes, persistent volume claims, horizontal pod autoscale in all namespaces
 
    ```kubectl get pods,deploy,svc,pv,pvc,hpa --all-namespaces -o wide```
- Delete pods, deployments, services, persistent volumes, persistent volume claims, horizontal pod autoscale all at once (Don't delete service/kubernetes)

    ```kubectl delete pod/php-776fc877d8-gp2fm service/php ``` (use NAME from ```kubectl get pods,deploy,svc,pv,pvc,hpa```)


## Firewall Exceptions for K3s

![image](https://user-images.githubusercontent.com/16585545/211191995-c511e9ce-33e5-4af7-a5f5-56dea769f172.png)

