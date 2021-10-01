# Deploy and Manage Cloud Environments with Google Cloud: Challenge Lab

## Task 1 : Create Production Environment
 #### Navigation Menu ->Compute engine ->VM instances 
 #### SSH to kraken-jumphost -> Run
 
```bash
cd /work/dm
sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network --config=prod-network.yaml

gcloud config set compute/zone us-east1-b

gcloud container clusters create kraken-prod \
          --num-nodes 2 \
          --network kraken-prod-vpc \
          --subnetwork kraken-prod-subnet\
          --zone us-east1-b

gcloud container clusters get-credentials kraken-prod

cd /work/k8s

for F in $(ls *.yaml); do kubectl create -f $F; done
```
## Task 2 : Setup the Admin instance
#### Still in kraken-jumphost's SSH -> Run

```bash
gcloud config set compute/zone us-east1-b

gcloud compute instances create kraken-admin --network-interface="subnet=kraken-mgmt-subnet" --network-interface="subnet=kraken-prod-subnet"
```
### Create alert

#### Open monitoring
#### Create an alert
#### Configure the policy to email your email and set
```bash
   Resource Type : VM Instance
   Metric : CPU utilization
   Filter : instance_name
            Value : kraken-admin
   Condition : is above
   Threshold : 50%
   For : 1 minute
```
## Task 3 : Verify the Spinnaker deployment
#### Switch to cloudshell, run
```bash
gcloud config set compute/zone us-east1-b

gcloud container clusters get-credentials spinnaker-tutorial

DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
```
#### Go to cloudshell webpreview

#### Go to applications -> sample

#### Open pipelines and manually run the pipeline if it has not already running.

#### Approve the deployment to production.

#### Check the production frontend endpoint (use http, not the default https)

#### Back to cloudshell, run to push a change

```bash
gcloud config set compute/zone us-east1-b

gcloud source repos clone sample-app

cd sample-app
touch a

git config --global user.email "$(gcloud config get-value account)"
git config --global user.name "Student"
git commit -a -m "change"
git tag v1.0.1
git push --tags
```
##### This lab is done 😊
