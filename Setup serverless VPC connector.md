## [SIK] Setup VPC Serverless connector

VPC name `apics-service-network` subnet `apics-logging-subnet`

1. Create VPC connector `orchestration-connector`

```bash
gcloud compute networks vpc-access connectors create orchestration-connector \
    --network apics-service-network \
    --region europe-west3 \
    --range 10.8.0.0/28
```

gcloud compute addresses create apics-static-ip \
    --region=europe-west3

gcloud compute routers create apics-router \
    --network apics-service-network \
    --region europe-west3

gcloud compute routers nats create apics-cloud-nat-config \
    --router=apics-router \
                --router-region europe-west3 \
    --nat-external-ip-pool=apics-static-ip \
    --nat-all-subnet-ip-ranges \
    --enable-logging

