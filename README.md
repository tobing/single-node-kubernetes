# Single Node Kubernetes
Single Node Kubernetes for Nginx & PHP-FPM with [K3s](https://k3s.io/)
> K3s is a highly available, certified Kubernetes distribution designed for production workloads in unattended, resource-constrained, remote locations or inside IoT appliances

## Installation

1. Install K3s using ```curl -sfL https://get.k3s.io | sh - ```

2. Check K3s installation ```kubectl get node```

3. Clone this repository: ```git clone https://github.com/tobing/single-node-kubernetes.git ``` 
    ```
    local_persistentvolume.yaml :   To define volume in the local host (/tmp)
    *_deployment.yaml           :   To define kubernetes Deployment object
    *_service.yaml              :   To define kubernetes Service
    *_hpa.yaml                  :   To define kubernetes Horizontal Pod Autoscaler
    nginx_configMap.yaml        :   To define config related to app (nginx.conf)
    ```

4. Traefik service installed and running automatically after K3s installation completed. Because Nginx service will use port 80 also, we need to delete Traefik service ```kubectl --namespace kube-system delete svc traefik```

5. Go to inside ```single-node-kubernetes``` directory and run ```kubectl apply -f .```

6. Verify all running properly ```kubectl get pods,deploy,svc,pv,pvc,hpa```

7. If something wrong check with ```kubectl describe <NAME>```, example
    - For pod       :   ```kubectl describe pod/php-776fc877d8-2q2fq```
    - For service   :   ```kubectl describe service/nginx```  
    
8. Take note the EXTERNAL-IP for ```service/nginx```

9. Open a web browser and go to http://EXTERNAL-IP. The page should show "403 Forbidden"

10. Because the persistent volume location in /tmp we need to create a file to test ```vi /tmp/test.php``` and type code below and save
    ```
    <?php
        echo gethostname();
    ?>
    ```
    
11. Back to the web browser and open http://EXTERNAL-IP/test.php. Reload the page several times and the hostname will changed. This mean requests served by different pod

12. To check Horizontal Pod AutoScaler (hpa) need to open 2 shell session
    - First shell to monitor the pods run command ```watch -n1 kubectl get pods```
    - Second shell from same or different server to run ab apache benchmark (change -n value accordingly)
        ```ab -n 100000 -c 1000 http://EXTERNAL-IP/test.php/```
        
    After the ab apache benchmark running for some time, you will see the pods replica increasing in the first shell

13. You can modify *hpa.yaml in the sections ```minReplicas```, ```maxReplicas```, ```averageUtilization```. After modified run ```kubectl apply -f .``` then test again and verify


## Firewall Exceptions for K3s

![image](https://user-images.githubusercontent.com/16585545/211191995-c511e9ce-33e5-4af7-a5f5-56dea769f172.png)

