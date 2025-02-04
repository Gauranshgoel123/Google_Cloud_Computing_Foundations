# GSP751 
[![](https://github.com/CodingWithHardik/CodingWithHardik/blob/main/img/subscribe_button.png)](https://www.youtube.com/@CloudHustlers)
## Run in cloudshell 
### Zone from task 2 >`Create the managed instance groups`> step 3
```cmd
export ZONE=
```
```cmd
export REGION=${ZONE::-2}
last_char="${ZONE: -1}"
if [ "$last_char" == "a" ]; then
    export NZONE="${ZONE%?}b"  
elif [ "$last_char" == "b" ]; then
    export NZONE="${ZONE%?}c" 
elif [ "$last_char" == "c" ]; then
    export NZONE="${ZONE%?}b"
elif [ "$last_char" == "d" ]; then
    export NZONE="${ZONE%?}b"
fi
gcloud compute firewall-rules create app-allow-http --network my-internal-app --action allow --direction INGRESS --target-tags lb-backend --source-ranges 0.0.0.0/0 --rules tcp:80
gcloud compute firewall-rules create app-allow-health-check --network default --action allow --direction INGRESS --target-tags lb-backend --source-ranges 130.211.0.0/22,35.191.0.0/16 --rules tcp
gcloud compute instance-templates create instance-template-1 --machine-type=e2-medium --network=my-internal-app --region $REGION --subnet=subnet-a --tags=lb-backend --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh
gcloud compute instance-templates create instance-template-2 --machine-type=e2-medium --network=my-internal-app --region $REGION --subnet=subnet-b --tags=lb-backend --metadata=startup-script-url=gs://cloud-training/gcpnet/ilb/startup.sh
gcloud compute instance-groups managed create instance-group-1 --base-instance-name=instance-group-1 --template=instance-template-1 --zone=$ZONE --size=1 
gcloud compute instance-groups managed set-autoscaling instance-group-1 --zone=$ZONE --cool-down-period=45 --max-num-replicas=5 --min-num-replicas=1 --target-cpu-utilization=0.8
gcloud compute instance-groups managed create instance-group-2 --base-instance-name=instance-group-2 --template=instance-template-2 --zone=$NZONE --size=1 
gcloud compute instance-groups managed set-autoscaling instance-group-2 --zone=$NZONE --cool-down-period=45 --max-num-replicas=5 --min-num-replicas=1 --target-cpu-utilization=0.8
gcloud compute instances create utility-vm --zone=$ZONE --machine-type=e2-micro --image-family=debian-10 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --network=my-internal-app --subnet=subnet-a --private-network-ip=10.10.20.50
gcloud compute health-checks create tcp my-ilb-health-check \
--description="Subscribe To CloudHustlers" \
--check-interval=5s \
--timeout=5s \
--unhealthy-threshold=2 \
--healthy-threshold=2 \
--port=80 \
--proxy-header=NONE
TOKEN=$(gcloud auth application-default print-access-token)
cat > 1.json <<EOF
{
    "backends": [
      {
        "balancingMode": "CONNECTION",
        "group": "projects/$DEVSHELL_PROJECT_ID/zones/$ZONE/instanceGroups/instance-group-1"
      },
      {
        "balancingMode": "CONNECTION",
        "group": "projects/$DEVSHELL_PROJECT_ID/zones/$NZONE/instanceGroups/instance-group-2"
      }
    ],
    "connectionDraining": {
      "drainingTimeoutSec": 300
    },
    "description": "",
    "healthChecks": [
      "projects/$DEVSHELL_PROJECT_ID/global/healthChecks/my-ilb-health-check"
    ],
    "loadBalancingScheme": "INTERNAL",
    "logConfig": {
      "enable": false
    },
    "name": "my-ilb",
    "network": "projects/$DEVSHELL_PROJECT_ID/global/networks/my-internal-app",
    "protocol": "TCP",
    "region": "projects/$DEVSHELL_PROJECT_ID/regions/$REGION",
    "sessionAffinity": "NONE"
  }
EOF
cat > 2.json <<EOF
{
   "IPAddress": "10.10.30.5",
   "loadBalancingScheme": "INTERNAL",
   "allowGlobalAccess": false,
   "description": "SUBSCRIBE TO CLOUDHUSTLER",
   "ipVersion": "IPV4",
   "backendService": "projects/$DEVSHELL_PROJECT_ID/regions/$REGION/backendServices/my-ilb",
   "IPProtocol": "TCP",
   "networkTier": "PREMIUM",
   "name": "my-ilb-forwarding-rule",
   "ports": [
     "80"
   ],
   "region": "projects/$DEVSHELL_PROJECT_ID/regions/$REGION",
   "subnetwork": "projects/$DEVSHELL_PROJECT_ID/regions/$REGION/subnetworks/subnet-b"
 }
EOF
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d  @1.json \
  "https://compute.googleapis.com/compute/v1/projects/$DEVSHELL_PROJECT_ID/regions/$REGION/backendServices"
sleep 20
curl -X POST -H "Content-Type: application/json" \
 -H "Authorization: Bearer $TOKEN" \
 -d @2.json \
 "https://compute.googleapis.com/compute/v1/projects/$DEVSHELL_PROJECT_ID/regions/$REGION/forwardingRules"
```
