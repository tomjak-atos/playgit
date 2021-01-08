## [SIK] Setup VPC Serverless connector

VPC name `apics-service-network` subnet `apics-logging-subnet`

1. Create VPC connector `orchestration-connector`

```bash
gcloud compute networks vpc-access connectors create orchestration-connector \
    --network apics-service-network \
    --region europe-west3 \
    --range 10.8.0.0/28
```
2. Request static IP

```bash
gcloud compute addresses create apics-static-ip \
    --region=europe-west3
```
3. Create router

```bash
gcloud compute routers create apics-router \
    --network apics-service-network \
    --region europe-west3
```

4. Create NAT

```bash
gcloud compute routers nats create apics-cloud-nat-config \
    --router=apics-router \
                --router-region europe-west3 \
    --nat-external-ip-pool=apics-static-ip \
    --nat-all-subnet-ip-ranges \
    --enable-logging
```

5. Now connector can be used when deploying Cloud Functions. Sample deploy:

```bash
# Configurate the connector
gcloud functions deploy listen \
    --runtime python37 \
    --entry-point listen \
    --trigger-http \
    --allow-unauthenticated \
    --vpc-connector orchestration-connector \
    --egress-settings all
```



## Notes and prerequisites

Test CF
main.py

```bash
# This function will return the IP address for egress
import requests
import json

def listen(request):
    result = requests.get("https://api.ipify.org?format=json")
    return json.dumps(result.json())
	
gcloud functions deploy listen \
    --runtime python37 \
    --entry-point test_ip \
    --trigger-http \
    --allow-unauthenticated
```
	
Initial VPC

```bash
# Create VPC
gcloud services enable compute.googleapis.com

gcloud compute networks create apics-service-network \
    --subnet-mode=custom \
    --bgp-routing-mode=regional
```

Make sure CF service account has permissions

```bash
# Grant Permissions 
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export PROJECT_NUMBER=$(gcloud projects list --filter="$PROJECT_ID" --format="value(PROJECT_NUMBER)")

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:service-$PROJECT_NUMBER@gcf-admin-robot.iam.gserviceaccount.com \
--role=roles/viewer

gcloud projects add-iam-policy-binding $PROJECT_ID \
--member=serviceAccount:service-$PROJECT_NUMBER@gcf-admin-robot.iam.gserviceaccount.com \
--role=roles/compute.networkUser
```