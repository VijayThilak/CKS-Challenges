# CKS-Challenges

# Challenge 1

# Scenario

<img width="979" alt="Screenshot 2023-03-10 at 8 28 45 AM" src="https://user-images.githubusercontent.com/8725714/224212235-86164a13-88f2-40fb-b83f-d52b1d849064.png">


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

Use 'trivy' to scan and find the nginx image with the least number of 'CRITICAL' vulnerabilities

```
for i in $(docker images | grep nginx | awk '{print $1 ":" $2}' | sort -hr) ; do echo $i ; trivy image --severity CRITICAL $i 2>&1 | grep Total ; echo ---- ; done

```

## TASK 4 - Modify the deployment 


1. Create a deployment  with the least 'CRITICAL' vulnerabilities image - nginx:alpine 

2. Deployment has exactly '1' ready replica

3. 'data-volume' is mounted at '/usr/share/nginx/html' on the pod

4. Add Apparmor profile



## TASK 5 - Create Network Policy


## TASK 6 - Expose the Deployment

```
kubectl expose deploy -n alpha alpha-xyz   --port 80 --target-port 80 --type ClusterIP  --name alpha-svc --dry-run=client -o yaml > alpha-svc.yaml
 
k apply -f alpha-svc.yaml 
 
k get svc -n alpha
```

# Result

<img width="975" alt="Screenshot 2023-03-10 at 7 35 59 AM" src="https://user-images.githubusercontent.com/8725714/224209964-50024511-994a-4bff-b404-b1659fec967d.png">

# Solution Video
https://user-images.githubusercontent.com/8725714/224311613-89be16a0-457e-42d3-ac7f-50d9f967b6b0.mp4


# CHALLENGE 2
# Scenario
# Tasks
1. Modify and Build Docker Image using Dockerfile
2. Scan the config files to fix security issues using kubesec
3. Remove shell access by using  startupProbe
4. Access Secret using environment variables within deployment
5. Implent Network Policy
# CHALLENGE 3
# Scenario
# CHALLENGE 4
# Scenario
