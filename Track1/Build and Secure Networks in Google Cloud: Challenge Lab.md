# GSP322 : Build and Secure Networks in Google Cloud: Challenge Lab

## Task 1 : Remove the overly permissive rules
```bash
gcloud compute firewall-rules delete open-access
```

## Task 2 : Start the bastion host instance

- **Go to Compute Engine and start Bastion instance**

## Task 3 : Create a firewall rule that allows SSH (tcp/22) from the IAP service and add network tag on bastion
```bash
gcloud compute firewall-rules create ssh-ingress --allow=tcp:22 --source-ranges 35.235.240.0/20 --target-tags ssh-ingress --network acme-vpc

gcloud compute instances add-tags bastion --tags=ssh-ingress --zone=us-central1-b
```
## Task 4 : Create a firewall rule that allows traffic on HTTP (tcp/80) to any address and add network tag on juice-shop
```bash
gcloud compute firewall-rules create http-ingress --allow=tcp:80 --source-ranges 0.0.0.0/0 --target-tags http-ingress --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=http-ingress --zone=us-central1-b
```
## Task 5 : Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address and add network tag on juice-shop
```bash
gcloud compute firewall-rules create internal-ssh-ingress --allow=tcp:22 --source-ranges 192.168.10.0/24 --target-tags internal-ssh-ingress --network acme-vpc

gcloud compute instances add-tags juice-shop --tags=internal-ssh-ingress --zone=us-central1-b
```
## Task 6 : SSH to bastion host via IAP and juice-shop via bastion
- **In Compute Engine -> VM Instances page, click the SSH button for the bastion host. Then SSH to juice-shop by**
```bash
ssh [Internal IP address of juice-shop]
```

**This Lab is Done**😊



