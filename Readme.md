### GKE deployment
### Set variables
```sh
PROJECT_ID=gopara-tf-sandbox-gke
CLUSTER_NAME=gopara-tf-sandbox
```

### Set environment 
```sh
gcloud config set project $PROJECT_ID
gcloud container clusters get-credentials $CLUSTER_NAME
```
### Create secret
```sh
kubectl create secret generic db-secret \ 
--from-literal=POSTGRES_CONN_STRING='postgresql://<db app user name>:<db app user password>@db/appdb' \ 
--from-literal=POSTGRES_PASSWORD='<postgres root password>'
```
## Database deployment
### Label one of the nodes with 'app=db' for db pod node affinity
```sh
node_name=$(kubectl get nodes -o jsonpath={.items[0].metadata.name})
kubectl label nodes $node_name app=db
```
### Deploy config map for db
```sh
kubectl apply -f ./k8s/db-cm.yaml
```
### Deploy persistent volume claim for db
```sh
kubectl apply -f ./k8s/db-pvc-gke.yaml
```
### Deploy service for db
```sh
kubectl apply -f ./k8s/db-svc.yaml
```
### Deploy stateful set for db
```sh
kubectl apply -f ./k8s/db-sts.yaml
```

### Deploy database appdb
```sh
kubectl exec -it db-statefulset-0 -- /bin/bash
psql -U postgres
```
```sql
CREATE DATABASE appdb;
CREATE USER appuser WITH PASSWORD '<put the appuser password here>';
GRANT ALL PRIVILEGES ON DATABASE appdb to appuser;
\c appdb
GRANT USAGE, CREATE ON SCHEMA public TO appuser; on appdb;
GRANT ALL ON SCHEMA public TO appuser
\q
```
```sh
exit
exit
```
## Application deployment
### Deploy config map for app
```sh
kubectl apply -f ./k8s/app-cm.yaml
```
### Deploy service for app
```sh
kubectl apply -f ./k8s/app-svc.yaml
```
### Deploy deployment for app
```sh
kubectl apply -f ./k8s/app-deploy.yaml
```
