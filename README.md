# CKS-Challenges

# Challenge-1

# Scenario

<img width="979" alt="Screenshot 2023-03-10 at 8 28 45 AM" src="https://user-images.githubusercontent.com/8725714/224212235-86164a13-88f2-40fb-b83f-d52b1d849064.png">


## Tasks

 1. PVC binding to PV
 2. Load Apparmor to secure deployment
 3. Image scanning by Trivy
 4. Create Network Policy
 5. Expose Deployment


## TASK 1 - PVC binding to PV

Persistent Volume alpha-pv  has already been created. Check the 'access modes' & 'capacity' of the PV and PVC. 
Here PV access mode - RWX (READWRITEMANY), PVC access mode - RWO (READWRITEINCE). Hence change the access mode of the pvc.
   
<img width="1145" alt="Screenshot 2023-03-10 at 8 43 37 AM" src="https://user-images.githubusercontent.com/8725714/224214466-a736a91e-4d4e-4045-b2b3-96cf501f979a.png">


<img width="1211" alt="Screenshot 2023-03-10 at 8 45 13 AM" src="https://user-images.githubusercontent.com/8725714/224214470-27af6028-832a-4d74-bdd0-e2d85e7c2705.png">



# Result

<img width="975" alt="Screenshot 2023-03-10 at 7 35 59 AM" src="https://user-images.githubusercontent.com/8725714/224209964-50024511-994a-4bff-b404-b1659fec967d.png">

# Solution Video

https://user-images.githubusercontent.com/8725714/224211215-ab58dbe1-ceea-415f-9d2a-b8d636509a59.mp4
