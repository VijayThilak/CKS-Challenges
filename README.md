# CKS-Challenges

# Challenge 1

# Scenario

<img width="964" alt="Screenshot 2023-03-16 at 1 12 57 PM" src="https://user-images.githubusercontent.com/8725714/225551840-4e1a80b0-ecf6-47ff-bb16-3a263efa5285.png">



# Tasks

 1. PVC binding to PV
 2. Load Apparmor profile to secure deployment
 3. Image scanning by Trivy
 4. Modify the deployment 
 5. Create Network Policy
 6. Expose the Deployment


## TASK 1 - PVC binding to PV

Persistent Volume `alpha-pv` has been created already. Access mode of  PV  `RWX (READWRITEMANY)` and PVC `RWO (READWRITEONCE)` are different here. Hence change the access mode of the pvc.

```
root@controlplane ~ ➜  k get pv,pvc -n alpha 
NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
persistentvolume/alpha-pv   1Gi        RWX            Delete           Available           local-storage            117s

NAME                              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS    AGE
persistentvolumeclaim/alpha-pvc   Pending                                      local-storage   117s 
```
   

Edit the parameters of PVC and ensure it is bounded to PV

```
k edit pvc -n alpha alpha-pvc 

k replace --force -f /tmp/kubectl-edit-235559513.yaml
```
```
root@controlplane ~ ➜  k get pv,pvc -n alpha
NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS    REASON   AGE
persistentvolume/alpha-pv   1Gi        RWX            Delete           Bound    alpha/alpha-pvc   local-storage            13m

NAME                              STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
persistentvolumeclaim/alpha-pvc   Bound    alpha-pv   1Gi        RWX            local-storage   55s

```
<img width="1211" alt="Screenshot 2023-03-10 at 8 45 13 AM" src="https://user-images.githubusercontent.com/8725714/224214470-27af6028-832a-4d74-bdd0-e2d85e7c2705.png">


## TASK 2 - Load Apparmor profile to secure deployment

```
root@controlplane ~ ➜  aa-status | grep custom

root@controlplane ~ ➜  mv /root/usr.sbin.nginx /etc/apparmor.d/

root@controlplane ~ ➜  apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx

root@controlplane ~ ➜  aa-status | grep custom
   custom-nginx
  
 ```

## TASK 3 - Image scanning by Trivy

Get all the nginx images 
```
docker images | grep nginx | awk '{print $1 ":" $2}' | sort -hr
```

Use `trivy` to scan and find the nginx image with the least number of `CRITICAL` vulnerabilities

```
for i in $(docker images | grep nginx | awk '{print $1 ":" $2}' | sort -hr) ; do echo $i ; trivy image --severity CRITICAL $i 2>&1 | grep Total ; echo ---- ; done

```

## TASK 4 - Modify the deployment 

[config ] (https://github.com/VijayThilak/CKS-Challenges/blob/main/CKS-Challenge1/alpha-xyz.yaml)


1. Create a deployment  with the least 'CRITICAL' vulnerabilities image - nginx:alpine 

2. Deployment has exactly '1' ready replica

3. 'data-volume' is mounted at '/usr/share/nginx/html' on the pod

4. Add Apparmor profile



## TASK 5 - Create Network Policy

external' pod should NOT be able to connect to 'alpha-svc' on port 80


Inbound access only allowed from the pod called 'middleware' with label 'app=middleware'



## TASK 6 - Expose the Deployment

Expose the deployment 'alpha-xyz' as a 'ClusterIP' type service called 'alpha-svc

```
kubectl expose deploy -n alpha alpha-xyz   --port 80 --target-port 80 --type ClusterIP  --name alpha-svc --dry-run=client -o yaml > alpha-svc.yaml
 
k apply -f alpha-svc.yaml 
 
k get svc -n alpha
```

# Result

<img width="969" alt="Screenshot 2023-03-16 at 1 21 05 PM" src="https://user-images.githubusercontent.com/8725714/225551983-261caa3f-1e95-4826-b919-895edca910fe.png">


# Solution Video
https://user-images.githubusercontent.com/8725714/224311613-89be16a0-457e-42d3-ac7f-50d9f967b6b0.mp4




# CHALLENGE 2
# Scenario

<img width="1331" alt="Screenshot 2023-03-16 at 3 37 37 PM" src="https://user-images.githubusercontent.com/8725714/225595963-106b53c5-6582-4b8f-b2ab-e3e927c7fd0e.png">



# Tasks
1. Modify and Build Docker Image using Dockerfile
2. Scan the config files to fix security issues using kubesec
3. Remove shell access by using  startupProbe
4. Access Secret using environment variables within deployment
5. Implent Network Policy


# Result

<img width="1331" alt="Screenshot 2023-03-16 at 4 15 38 PM" src="https://user-images.githubusercontent.com/8725714/225595970-ce59bdc4-d8f8-4f5f-a2b4-e4f31dc27914.png">


# CHALLENGE 3
# Scenario
<img width="1266" alt="Screenshot 2023-03-16 at 12 00 55 PM" src="https://user-images.githubusercontent.com/8725714/225533923-6c9f44b2-ca68-4b94-9036-d35babf52581.png">

# Result
<img width="1274" alt="Screenshot 2023-03-16 at 12 21 21 PM" src="https://user-images.githubusercontent.com/8725714/225538916-7316f363-76ec-4fee-a385-2f73bfacd36a.png">

# CHALLENGE 4
# Scenario
<img width="930" alt="Screenshot 2023-03-16 at 5 48 15 PM" src="https://user-images.githubusercontent.com/8725714/225626184-7686ee57-e2bb-4e43-9a16-4db2154023a3.png">
# Result
<img width="929" alt="Screenshot 2023-03-16 at 6 36 36 PM" src="https://user-images.githubusercontent.com/8725714/225626201-79af5b52-955f-490e-a2a4-c12402afc613.png">
