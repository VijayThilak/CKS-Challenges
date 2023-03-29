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
k get pv,pvc -n alpha 

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
k get pv,pvc -n alpha
NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS    REASON   AGE
persistentvolume/alpha-pv   1Gi        RWX            Delete           Bound    alpha/alpha-pvc   local-storage            13m

NAME                              STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
persistentvolumeclaim/alpha-pvc   Bound    alpha-pv   1Gi        RWX            local-storage   55s

```
<img width="1211" alt="Screenshot 2023-03-10 at 8 45 13 AM" src="https://user-images.githubusercontent.com/8725714/224214470-27af6028-832a-4d74-bdd0-e2d85e7c2705.png">


## TASK 2 - Load Apparmor profile to secure deployment

Move AppArmor profile to given location on controlplane node. 
```
mv /root/usr.sbin.nginx /etc/apparmor.d/
```

Load and Enforce AppArmor profile `custom-nginx`
```
apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx
```

To Ensure the apparmor profile is Loaded
```
aa-status | grep custom
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




1. Create a [Deployment](https://github.com/VijayThilak/CKS-Challenges/blob/main/CKS-Challenge1/alpha-xyz.yaml)  with the least 'CRITICAL' vulnerabilities image - `nginx:alpine` 

2. Deployment has exactly '1' ready replica

3. `data-volume` is mounted at `/usr/share/nginx/html` on the pod

4. Add Apparmor profile



## TASK 5 - Create Network Policy


External pod should NOT be able to connect to `alpha-svc` on port 80 [External network policy](https://github.com/VijayThilak/CKS-Challenges/blob/main/Challenge1/external-netpol.yaml)


Inbound access only allowed from the pod called `middleware` with label `app=middleware` [Restrict network policy](https://github.com/VijayThilak/CKS-Challenges/blob/main/Challenge1/restrict-inbound.yaml)



## TASK 6 - Expose the Deployment

Expose the deployment `alpha-xyz` as a `ClusterIP` type service called [alpha-svc](https://github.com/VijayThilak/CKS-Challenges/blob/main/Challenge1/alpha-svc.yaml)

```
kubectl expose deploy -n alpha alpha-xyz   --port 80 --target-port 80 --type ClusterIP  --name alpha-svc --dry-run=client -o yaml > alpha-svc.yaml
 
k apply -f alpha-svc.yaml 
 
k get svc -n alpha
```

# Result

<img width="969" alt="Screenshot 2023-03-16 at 1 21 05 PM" src="https://user-images.githubusercontent.com/8725714/225551983-261caa3f-1e95-4826-b919-895edca910fe.png">


# Solution Video
https://user-images.githubusercontent.com/8725714/224311613-89be16a0-457e-42d3-ac7f-50d9f967b6b0.mp4




# Challenge 2
# Scenario

<img width="1331" alt="Screenshot 2023-03-16 at 3 37 37 PM" src="https://user-images.githubusercontent.com/8725714/225595963-106b53c5-6582-4b8f-b2ab-e3e927c7fd0e.png">



# Tasks
1. Modify and Build Docker Image using Dockerfile
2. Scan the config files to fix security issues using kubesec
3. Remove shell access by using  startupProbe
4. Access Secret using environment variables within deployment
5. Implement Network Policy


## Task 1 - Modify and Build Docker Image using Dockerfile


Move the required files and directories (app.py, requirements.txt and the templates directory) to a subdirectory called `app` under `webapp` and update the COPY instruction in the [Dockerfile](https://github.com/VijayThilak/CKS-Challenges/blob/main/Challenge2/Dockerfile) accordingly

```
docker build -t kodekloud/webapp-color:stable .
```
## Task 2 - Scan the config files to fix security issues using kubesec

```
kubesec scan staging-webapp.yaml

image: kodekloud/webapp-color:stable
    imagePullPolicy: Never
    name: webapp-color
    resources: {}
    securityContext:
      runAsUser: 0
      allowPrivilegeEscalation: false
      capabilities:
        add:
        - NET_ADMIN       
```

## Task 3 - Remove shell access by using  startupProbe

Redeploy the 'staging-webapp' pod once issues are fixed 

## Task 4 - Create and access the Secret using environment variables within deployment

Create a secret called `prod-db` for all the hardcoded values and consume the secret values as environment variables within the deployment.

```
k get deployments.apps prod-web -n prod -oyaml > prod-web.yaml

cat prod-web.yaml | grep -A6 "env:"

kubectl create secret generic prod-db --from-literal DB_Host=prod-db --from-literal DB_User=root --from-literal DB_Password=paswrd -n prod

k describe secrets prod-db 

k edit deployments.apps prod-web -n prod

k get pod -n prod

```

## Task 5 - Implement Network Policy

Create a network policy called `net-po` that will only allow traffic only within the `prod` namespace and deny all the traffic from other namespaces.

# Result

<img width="1331" alt="Screenshot 2023-03-16 at 4 15 38 PM" src="https://user-images.githubusercontent.com/8725714/225595970-ce59bdc4-d8f8-4f5f-a2b4-e4f31dc27914.png">

# Solution Video

https://user-images.githubusercontent.com/8725714/227878140-89c56152-4c7b-4fa1-866e-ef3d40877216.mp4


# Challenge 3

# Scenario
<img width="1266" alt="Screenshot 2023-03-16 at 12 00 55 PM" src="https://user-images.githubusercontent.com/8725714/225533923-6c9f44b2-ca68-4b94-9036-d35babf52581.png">

# Tasks

1. Run `kube-bench` to identify and fix issues related to controlplane and work node components
2. Fix kube-apiserver auditing issues
3. Fix kubelet security issues
4. Fix etcd, kube-controller-manager and kube-scheduler security issues


## Task 1 - Install and run `kube-bench` to find and fix issues on Controlplane & Worker node

Install [kube-bench](https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md) on `controlplane`
```
cd /opt
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.2/kube-bench_0.6.2_linux_amd64.tar.gz -o kube-bench_0.6.2_linux_amd64.tar.gz
tar -xvf kube-bench_0.6.2_linux_amd64.tar.gz
```
Run `kube-bench` to find the issues

```
mkdir -p /var/www/html
./kube-bench run --config-dir /opt/cfg --config /opt/cfg/config.yaml > /var/www/html/index.html
```


## Task 2 - Fix  the kube-apiserver  issues

Check the kubelet config file to find the manifests path

```
ps -ef |grep kubelet |grep config

/var/lib/kubelet/config.yaml

cat /var/lib/kubelet/config.yaml

$$$ staticPodPath: /etc/kubernetes/manifests

````

Edit the kube-apiserver manifests as per kube-bench results

```
vim /etc/kubernetes/manifests/kube-apiserver.yaml
parameters:

- --profiling=false
- --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
- --insecure-port=0
- --audit-log-path=/var/log/apiserver/audit.log
- --audit-log-maxage=30
- --audit-log-maxbackup=10
- --audit-log-maxsize=100

  - mountPath: /var/log/apiserver/
    name: audit-log
    readOnly: false

  - name: audit-log
    hostPath:
      path: /var/log/apiserver/
      type: DirectoryOrCreate
    
```
Ensure the apiserver is up and running.
```
crictl ps -a
```

## Task 3 - Fix the kubelet issue

Modify the kubelet file on controlplane 
```
ps -ef |grep kubelet
vim /var/lib/kubelet/config.yaml 

protectKernelDefaults: true
```

Login to Workernode and modify the kubelet file
```
k get nodes
ssh node01

vim /var/lib/kubelet/config.yaml 

protectKernelDefaults: true
```

Run the kubelet restart on Controlplane and Workernode
```
systemctl daemon-reload
systemctl restart kubelet
```

## Task 4 - Fix etcd, kube-controller-manager and kube-scheduler issues


Fix Kube Controller Manager issue - Set the `profiling` parameter on kube-controller file
```
vim /etc/kubernetes/manifests/kube-controller-manager.yaml 
- --profiling=false
```

Fix ETCD issue - Ensure that the ownership of etcd data directory is set to etcd:etcd
```
ls -ltr /var/lib/ | grep etcd

chown etcd:etcd /var/lib/etcd

````



Fix Kube Scheduler issue - Set the `profiling` parameter on kube-scheduler file
```
vim /etc/kubernetes/manifests/kube-scheduler.yaml 
- --profiling=false
```

Once the changes done, kube-controller-manager and kube-scheduler pods will recreate. Ensure both pod are up and running
```
crictl ps -a
```

# Result
<img width="1274" alt="Screenshot 2023-03-16 at 12 21 21 PM" src="https://user-images.githubusercontent.com/8725714/225538916-7316f363-76ec-4fee-a385-2f73bfacd36a.png">


# Solution Video

https://user-images.githubusercontent.com/8725714/228216702-97c98fef-2bd9-4148-93d8-8fb92ef4c4ec.mp4


# Challenge 4
# Scenario
<img width="930" alt="Screenshot 2023-03-16 at 5 48 15 PM" src="https://user-images.githubusercontent.com/8725714/225626184-7686ee57-e2bb-4e43-9a16-4db2154023a3.png">

# Tasks

## Tasks 1 - Create an audit policy 

Check the roles,rolebinding and configmaps in citadel namespace.

```
k get all -n citadel
k get role -n citadel
k get cm -n citadel
k get rolebinding -n citadel
```

Then create a rule in the [audit Policy](https://github.com/VijayThilak/CKS-Challenges/blob/main/Challenge4/audit-policy.yaml) at `/etc/kubernetes/audit-policy.yaml` as per requirements



## Tasks 2 - Enable auditing in kube-apiserver

Modify the [kube-apiserver](https://github.com/VijayThilak/CKS-Challenges/blob/main/Challenge4/kube-apiserver.yaml) file `/etc/kubernetes/manifests/kube-apiserver.yaml` as below config

```
- --audit-policy-file=/etc/kubernetes/audit-policy.yaml
- --audit-log-path=/var/log/kubernetes/audit/audit.log

    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit/
      name: audit-log
      readOnly: false

  - name: audit
    hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
  - name: audit-log
    hostPath:
      path: /var/log/kubernetes/audit/
      type: DirectoryOrCreate
```
Ensure the kube-apiserver is  running

```
crictl ps -a
journalctl | grep apiserver
```

## Tasks 3 - Install and Configure Falco

Install [Falco](https://falco.org/docs/getting-started/installation/) and start it's service

```
cat /etc/*release*

FALCO_FRONTEND=noninteractive apt-get install -y falco

systemctl status kubelet 

systemctl start falco
systemctl status falco
```
Edit the Falco config file to save event logs at `/opt/falco.log`

```
vim /etc/falcofalco.yaml 
file_output:
  enabled:  true
  keep_alive: false
  filename: /opt/falco.log
```
To apply the change restart `Falco` service 

```
systemctl restart falco
systemctl status falco
```



## Tasks 4 - Delete the role and rolebinding causing the constant deletion and creation of the configmaps and pods 

Check the API server audit logs and find  the user responsible for the abnormal behaviour in the 'citadel' namespace
```
cat /var/log/kubernetes/audit/audit.log |grep citadel |grep -v "get\|watch\|list" |jq
```

Find the name of the `'user', 'role' and 'rolebinding'` responsible for the issue
```
k get sa -n citadel
k get role -n citadel
k get role important_role_do_not_delete -n citadel
```
```
k get rolebindings.rbac.authorization.k8s.io -n citadel
k get rolebindings.rbac.authorization.k8s.io important_binding_do_not_delete -n citadel
```
```
k get rolebindings.rbac.authorization.k8s.io important_binding_do_not_delete -n citadel -oyaml
``` 
Save the name of the `'user, 'role' and 'rolebinding'` responsible for the event to the file `'/opt/blacklist_users'` file

```
echo "agent-smith,important_role_do_not_delete,important_binding_do_not_delete" > /opt/blacklist_users
```

Inspect the `falco` logs to find the pod that has events generated because of packages being updated on it
```
Journalctl -f falco.log 

Mar 15 13:40:26 controlplane falco[23510]: 13:40:26.974065686 Error Package management process launched in container (user=root user_loginuid=-1 command=apt install nginx container_id=c3c7bfd6dd6e  container_name=k8s_eden-software2_eden-software2_eden-prime_ 94744b4a-a073-490b-8bef-f54c0313e873_0  image=ubuntu:latest)
```

Find the container ID
```

crictl ps |grep "c3c7bfd6dd6e"
```

Find the pod namespace and name
```
crictl pods| grep "eden-software2"
c3c7bfd6dd6e       12 minutes ago       Ready               eden-software2                         eden-prime          0                   (default)
```  

Save the namespace and pod name to file `/opt/compromised_pods`
```
echo "eden-prime,eden-software2" > /opt/compromised_pods
```


Delete the Compromised pods in the `eden-prime` namespace 
```
k delete pod eden-software2 -n eden-prime --force
```
Find the role and rolebinding causing the constant deletion and creation of the configmaps and pods in `citadel` namespace
```
k get role -n citadel
k get rolebinding -n citadel
``` 

Delete the role and rolebindings 
```
k delete role important_role_do_not_delete -n citadel
k delete rolebinding important_binding_do_not_delete -n citadel 
```


# Result
<img width="929" alt="Screenshot 2023-03-16 at 6 36 36 PM" src="https://user-images.githubusercontent.com/8725714/225626201-79af5b52-955f-490e-a2a4-c12402afc613.png">

# Solution Video
https://user-images.githubusercontent.com/8725714/228230587-f7ed415d-ac7b-4965-a2b6-b62f73451efb.mp4


                                                                # THANK YOU
