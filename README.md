# Single Node Kubernetes for Nginx & PHP-FPM
Single Node Kubernetes for Nginx & PHP-FPM with [K3s](https://k3s.io/)  ```Tested in Ubuntu 22.04 and Rocky Linux 8.5 RAM 2GB+```
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

    ![image](https://user-images.githubusercontent.com/16585545/211205542-0f97d2f9-01a4-4501-998b-55c429ff98c5.png)

    
4. Traefik service installed and running automatically after K3s installation completed. Because Traefik & Nginx service using port 80, we need to <b>DISABLE</b> it first 
    - Check Traefik using port 80 ```kubectl get svc -n=kube-system```
    - Stop K3s service ```service k3s stop```
    - Modify ```/etc/systemd/system/k3s.service``` by adding ```--disable=traefik \``` in the end of ExecStart 
    
        ```
        ExecStart=/usr/local/bin/k3s \
        server \
        --disable=traefik \
        ```
    - Reload for service config change ```systemctl daemon-reload```
    - Start K3s service ```service k3s start```
    - Check Traefik service not running ``````kubectl get svc -n=kube-system``````

    >Or <b>DELETE</b> the service ```kubectl --namespace kube-system delete svc traefik``` (not recommended)
   
   &nbsp;
    
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

&nbsp;
&nbsp;

# Extend to multi node by adding a worker node
```This worker node tested in Ubuntu 22.04 RAM 1GB+```

> **Note**
> Make sure docker engine installed and the hostname is not same with master node hostname

> **Note**
> Local persistent volume [ReadWriteOnce](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) access mode (see step 7 below for the impact) 


## Installation for worker node

1. Run this command in the worker node ```curl -sfL https://get.k3s.io | K3S_URL=https://MASTERNODEIP:6443 K3S_TOKEN=MASTERNODETOKEN sh - ```

   MASTERNODETOKEN stored at ```/var/lib/rancher/k3s/server/node-token```
   
2. Run ```kubectl get node``` in the master node to check
   ![image](https://user-images.githubusercontent.com/16585545/211222616-521bf3d3-8aed-44b7-8c49-7f27b5a7f14d.png)

3. Before adding the worker node to persistent volume, we need to stop all by running ```kubectl delete deployment.apps/nginx deployment.apps/php service/php service/nginx persistentvolume/local-pv persistentvolumeclaim/local-pv-claim horizontalpodautoscaler.autoscaling/nginx-hpa horizontalpodautoscaler.autoscaling/php-hpa```
    - If we don't stop all of them we will get ```The PersistentVolume "local-pv" is invalid: nodeAffinity:  field is immutable```
    - We cannot just stop PV because used by others, so need to stop all
4. Add the worker node name to ```local_persistentvolume.yaml```

    ![image](https://user-images.githubusercontent.com/16585545/211226295-009a2e30-20c3-4d2e-acc1-8ea755e6c25f.png)

5. Start all by run ```kubectl apply -f .```

6. Check by command ```kubectl get pod -o wide```. We will see pods running in both master and worker node

    ![image](https://user-images.githubusercontent.com/16585545/211226562-f742042a-d370-46d7-9a79-227ac0539127.png)

7. Open a web browser and go to http://EXTERNAL-IP/test.php. See step 11 master node installation. 
   
   Reload the page several times and we will see the hostname or "404 not found" error message. "404 not found" because the request served by pod in the worker node      and test.php is not exist in the /tmp of worker node
   
8. Follow step 10 master node installation by creating test.php in the /tmp of worker node and reload the page several times. 
   
   We will see the hostname properly without "404 not found". Because of this, we realize about [local persistent volume](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner) impact for multi node (Recommended to use NFS or cloud storage for multi node)
