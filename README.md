## Setup

### **NOTE:** You will need a gcloud account to run this.

```bash
git clone https://github.com/victordickson/assessment

cd assessment

sudo chmod +x setup.sh

./setup.sh

gcloud services enable container.googleapis.com

gcloud container clusters create fancy-cluster --num-nodes 2

sudo chmod +x deploy-monolith.sh

./deploy-monolith.sh

export GOOGLE_CLOUD_PROJECT=<project-id>

cd ~/microservices/src/orders

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0 .

kubectl create deployment orders --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0

kubectl expose deployment orders --type=LoadBalancer --port 80 --target-port 8081

cd ~/microservices/src/products

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0 .

kubectl create deployment products --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0

kubectl expose deployment products --type=LoadBalancer --port 80 --target-port 8082
```
### Get public ips of product and order services
```text
kubectl get service 
```
### Replace Order and Product IP address and reconfigure monolith
```bash
cd ~/monolith-to-microservices/react-app

vim .env.monolith

REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products

npm run build:monolith

cd ~/monolith

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .

kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0
```

## Migrate frontend to microservice
```bash
cd ~/react-app

cp .env.monolith .env

cd ~/microservices/src/frontend

gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0 .

kubectl create deployment frontend --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0

kubectl expose deployment frontend --type=LoadBalancer --port 80 --target-port 8080
```
### Test the microservice
```text
kubectl get service frontend
```
## Implement Monitoring using Prometheus 
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

kubectl create -n monitoring

helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

kubectl get all -n monitoring
```
### Visualise with grafana dashboard
```text
kubectl port-forward service/monitoring-grafana 8080:80 -n monitoring
```
### Default credentials; Username: admin , Password: prom-operator
